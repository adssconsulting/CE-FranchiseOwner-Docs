# Architecture

!!! warning "Internal — for the system administrator"
    This page is for the CE FranchiseOwner administrator (the franchise's
    technical owner). If you're a franchise owner just here to learn the app,
    head back to the [User Guide](../marketing/feature-guides.md).

CE FranchiseOwner is a **web application**: a FastAPI + Postgres backend and a
React/Vite single-page frontend, run as two Docker Compose stacks (development
and production) on one machine.

## Tech stack

**Backend**

- FastAPI 0.115 + Uvicorn 0.34 (single worker, no auto-reload)
- SQLAlchemy 2.0 (tables reflected via automap — no hand-written ORM that can
  drift from the DB), psycopg2-binary 2.9
- Pydantic 2.10 / pydantic-settings 2.7 for config
- Alembic 1.14 for migration tracking
- rapidfuzz 3.10 (fuzzy agent matching), openpyxl 3.1 (Excel), pdfplumber 0.11 +
  beautifulsoup4 (document/email parsing), anthropic 0.10x (LLM proxy),
  cryptography 44 (field encryption), pytest 8.3 (tests)

**Database:** PostgreSQL 16.

**Frontend:** React 18.3, react-router-dom 6.28, react-window (virtualized lists),
built with Vite 6. Plain JavaScript/JSX — no TypeScript.

## Folder structure

```
backend/
  app/
    main.py            # FastAPI app: CORS, login-gate middleware, startup hooks, /health
    config.py          # Settings (env vars only); derives env/is_prod from the DB name
    db.py              # Engine/session; automap reflection; get_db sets per-request RLS vars
    demo_auth.py       # Auth: HMAC tokens, PBKDF2 hashing, app_login accounts, territory scoping
    api.py             # Main API router (~3,600 lines) — all endpoint groups
    update_website.py  # Separate router for the Update Website FAQ tool (Anthropic proxy)
    queries.py         # SQL helper utilities (q.rows, q.val)
    email_parser.py / pdf_extractor.py / field_parsers.py / gate_check.py
                       # Application-intake parsing + pre-underwriting gate checks
    crypto.py          # Field-level encryption helpers
  demo/                # Ordered SQL bootstrap/migration scripts (00..07, rename_tables.sql)
  alembic/             # Migration scaffolding (0001_baseline = no-op stamp of the prod dump)
  tests/test_suite.py  # pytest suite run end-to-end against the live backend
frontend/
  src/
    main.jsx           # entry      App.jsx  # router shell      auth.jsx  # auth context
    pages/             # one component per screen (Dashboard, EmailQueue, ReviewMatches, …)
seed/ce_pg.sql         # (prod) production dump used to materialize the schema
docker-compose.yml     # parameterized by each folder's .env
deploy.ps1             # (prod) deploy script
```

## How it starts

- **Backend** runs under one Uvicorn worker (port 8000 in-container). On
  `@app.on_event("startup")` it (1) calls `seed_logins()` to upsert the hardcoded
  `demo_*` and `*_manager` accounts into `app_login` (never overwriting an existing
  password hash), and (2) starts the background unsubscribe/bounce inbox poller if
  enabled.
- **Frontend** runs the Vite dev server with hot-module reload; `VITE_API_BASE`
  points it at the backend.

## Authentication

- **Tokens** are HMAC-SHA256–signed usernames: `username|<hmac(secret, username)>`.
  Auth is a bearer token in the `Authorization` header (not cookies); CORS runs
  with `allow_credentials=False`. The signing secret is `DEMO_TOKEN_SECRET`.
- **Passwords** use salted PBKDF2-HMAC-SHA256 (200,000 iterations), stored as
  `pbkdf2_sha256:<iters>:<salt_b64>:<hash_b64>` — colon-delimited on purpose so a
  `$` never triggers docker-compose interpolation. Verified with `hmac.compare_digest`.
- **Account sources** are merged in `_accounts()`: hardcoded `DEMO_ACCOUNTS` (shared
  password `demo2026!`), hardcoded `PROD_ACCOUNTS` (`*_manager`, hashes from env),
  and the data-driven **`app_login`** table (cached ~20s). A DB password hash wins
  over a code/env hash, so onboarded franchises and admin password resets take effect.
- **Demo vs manager:** `demo_*` accounts authenticate with the shared `demo2026!`
  password; `*_manager` accounts authenticate against a stored hash. In **dev**, a
  hashless `*_manager` falls back to the shared demo password (fake data); in
  **prod**, a hashless `*_manager` is inert.
- **National admin** is **`demo_admin` in dev only** (the auth layer refuses the demo
  admin via the shared password when `is_prod`), and **`national_manager` in prod**
  (a real hashed password). This keeps the most-privileged account unreachable with
  the demo password on real data.
- A **login-gate middleware** returns 401 for any `/api/*` request without a valid
  identity, except an allowlist (`/api/auth/login`, `/api/environment`,
  `/api/auth/me`, and OPTIONS preflight).

## Multi-territory Row-Level Security (RLS)

Branch isolation is enforced by Postgres itself, not just application code.

- The app connects as the **non-superuser role `ce_app`** (`NOSUPERUSER NOBYPASSRLS`)
  so policies apply. The `ce_web`/`postgres` superusers (used for migrations/admin)
  bypass RLS.
- `get_db` derives `{territory, is_admin}` from the bearer token, sanitizes the
  territory, and sets the session variables `app.territory` and `app.is_admin` for
  the request. Every query is then auto-filtered.
- Two policy families:
    - **`terr_isolation`** (data tables — property loads, agent matches, email send
      log, master agent list, load batches, unsubscribe list, bounces, call-tracker
      matches, engagement): rows are visible when `is_admin='1'` **OR**
      `territory = app.territory`. The national admin sees all branches.
    - **`terr_config`** (the two config tables — `admin_app_settings`,
      `agentoutreach_email_template_dim`): rows require
      `territory = COALESCE(app.territory,'SC')`, with **no admin bypass**, so each
      branch edits only its own config. Unique keys are `(setting_key, territory)`
      and `(template_number, territory)`.
- **`app_login` is global with no RLS** — authentication happens before a territory
  is known, so the lookup must see every row.

See the [Database Schema](database-schema.md) for the per-table policy map.

## The two Docker stacks

The same `docker-compose.yml` runs twice, parameterized by each folder's `.env`.
The project name namespaces containers and volumes so both stacks run side by side.

| | Dev (`CE-FranchiseOwner-WEB-DEV`) | Prod (`CE-FranchiseOwner-WEB`) |
|---|---|---|
| `APP_ENV` | `dev` | `prod` |
| Backend port | 8001 | 8002 |
| Frontend port | 5173 | 5174 |
| Postgres port | 5433 | 5432 |
| Database | `ce_franchiseowner_web_dev` | `ce_franchiseowner` |
| Data | fake demo / multi-branch | real South Carolina data |

Each stack has three services: `postgres` (postgres:16 with a named volume +
healthcheck), `backend` (bind-mounts `./backend:/app`, connects as `ce_app`), and
`frontend` (bind-mounts `./frontend:/app` with Vite HMR). See
[Deployment](deployment.md) for the release flow.
