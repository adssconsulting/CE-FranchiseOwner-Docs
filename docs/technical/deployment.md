# Deployment Guide

!!! warning "Internal — for the system administrator"
    This page is for the CE FranchiseOwner administrator (the franchise's
    technical owner). If you're a franchise owner just here to learn the app,
    head back to the [User Guide](../marketing/feature-guides.md).

CE FranchiseOwner is a web app deployed as **two Docker Compose stacks** on one
machine — development and production — from two git checkouts. There is no
Windows `.exe`.

## The two stacks

| | Dev (`C:\Projects\CE-FranchiseOwner-WEB-DEV`) | Prod (`C:\Projects\CE-FranchiseOwner-WEB`) |
|---|---|---|
| Git branch | `development` | `main` |
| `APP_ENV` | `dev` | `prod` |
| Backend | `http://localhost:8001` | `http://localhost:8002` |
| Frontend | `http://localhost:5173` | `http://localhost:5174` |
| Postgres | port 5433 | port 5432 |
| Database | `ce_franchiseowner_web_dev` | `ce_franchiseowner` |
| Data | fake demo / multi-branch | real South Carolina data |

The same `docker-compose.yml` runs in each folder, parameterized by that folder's
`.env`; `COMPOSE_PROJECT_NAME` namespaces containers/volumes so both run side by side.

## Working in dev

- **Backend changes:** the backend is bind-mounted but Uvicorn runs with **no
  auto-reload** — run `docker compose restart backend` for Python changes to take effect.
- **Frontend changes:** Vite HMR picks them up live — no restart, no rebuild.
- **Dependency changes** (`requirements.txt` / `package.json` / a Dockerfile) require
  a rebuild: `docker compose up -d --build`.

## Releasing to production

Always: **fix in dev → verify → ship to prod.**

```bash
# 1. push dev work
git push origin development
# 2. fast-forward main to match
git push origin development:main
# 3. in the PROD folder, run the deploy script
powershell -NoProfile -ExecutionPolicy Bypass -File "C:/Projects/CE-FranchiseOwner-WEB/deploy.ps1"
```

`deploy.ps1` (refuses to run unless the current branch is `main`):

1. `git pull --ff-only origin main`
2. `docker compose up -d` (reuses images; pass `-Build` only when a dependency/Dockerfile changed)
3. `docker compose restart backend` (Uvicorn has no auto-reload)
4. Health-check: polls `http://localhost:8002/health` up to 10× (2s apart) for HTTP 200,
   then reports the app at `http://localhost:5174`.

## Environment variables

Set in each folder's `.env` (gitignored). Key ones (`.env.example` / `config.py`):

| Var | Purpose |
|---|---|
| `COMPOSE_PROJECT_NAME` | namespaces containers/volumes per env (`ce-web-dev` / `ce-web`) |
| `APP_ENV` | `dev` \| `prod`; drives the PROD banner and disables demo logins/admin in prod |
| `POSTGRES_USER` / `POSTGRES_PASSWORD` / `POSTGRES_DB` / `POSTGRES_HOST_PORT` | Postgres container + host port |
| `APP_DB_USER` / `APP_DB_PASSWORD` | the non-superuser RLS role (`ce_app` / `ce_app_pw`); compose builds `DATABASE_URL` from these |
| `DATABASE_URL` | full SQLAlchemy URL (compose composes the real one for the container) |
| `BACKEND_PORT` / `FRONTEND_PORT` / `VITE_API_BASE` / `CORS_ORIGINS` | ports + frontend/back wiring |
| `DEMO_TOKEN_SECRET` | HMAC token-signing secret (machine-local, never committed) |
| `SC_MANAGER_PW_HASH`, `NEWMEXICO_MANAGER_PW_HASH`, `NORTHCAROLINA_MANAGER_PW_HASH` | PBKDF2 hashes for the real `*_manager` accounts |
| `ANTHROPIC_API_KEY` | Update Website LLM proxy (server-side only) |
| `UNSUB_POLL_ENABLED` / `UNSUB_POLL_INTERVAL_SECONDS` / `UNSUB_POLL_TERRITORIES` / `CONTRACT_DAYS` | inbox poller (on in prod; default 900s, `SC`, 50 days) |

!!! note
    SMTP/IMAP credentials are **not** env vars — they live in `admin_app_settings`
    (`smtp.*`, `imap_*`), set through the Admin UI.

## Schema & migrations

- The base schema is materialized by loading the production dump `seed/ce_pg.sql`
  (~30 tables) into Postgres. Alembic's `0001_baseline` is a no-op marker so the DB
  can be `alembic stamp`-ed to a known starting point; future changes build forward.
- `backend/demo/*.sql` are ordered bootstrap/migration scripts, run by a superuser
  via `docker exec … psql`:

| Script | Purpose |
|---|---|
| `rename_tables.sql` | rename legacy desktop tables to the new convention |
| `00_prod_init_sc.sql` | prod-only: tag rows `territory='SC'`, give config tables the `(key, territory)` shape, create the national-report table |
| `01_territory_setup.sql` | demo: add `territory`, mark `SC`, duplicate into `DALLAS`/`VIRGINIA` |
| `02_rls_setup.sql` | create the `ce_app` non-superuser role, grants, enable RLS + `terr_isolation` |
| `03_*` / `04_*` | demo data variation + engagement |
| `05_per_territory_config.sql` / `05_email_bounce.sql` | per-territory config (demo) + bounce table |
| `06_app_login.sql` | the global, RLS-free `app_login` table (data-driven logins) |
| `07_config_territory.sql` | prod-safe `terr_config` RLS on the two config tables (no fake-branch duplication) |

Prod rebuild order: `seed/ce_pg.sql` → `rename_tables.sql` → `00` → `02` → `06` → `07`.

Reminder: after any backend code change, `docker compose restart backend`.
