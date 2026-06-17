# API & Integrations

!!! warning "Internal — for the system administrator"
    This page is for the CE FranchiseOwner administrator (the franchise's
    technical owner). If you're a franchise owner just here to learn the app,
    head back to the [User Guide](../marketing/feature-guides.md).

Every external system the app talks to, and where its config/keys come from. The
rule throughout: **secrets live server-side** — environment variables or the
`admin_app_settings` table — and never reach the browser.

## External integrations

### PropStream (property exports)
PropStream Excel exports are the primary property/agent feed. They're imported via
`POST /api/properties/import` (openpyxl) into `agentoutreach_property_load_fact`.
The `mls_agent_name` / `mls_agent_email` / `mls_brokerage_*` columns drive agent
matching (rapidfuzz) against the master list. No external API or key — it's a file
upload. PropStream login credentials for the scraper are stored in
`admin_app_settings` (keys `propstream.*`).

### MLS RESO Web API (OData)
`POST /api/properties/import-api` pulls Property records from a RESO/OData feed via
`_fetch_reso_properties` (dry-run "Pre-Check" or real import). The base URL comes
from `admin_app_settings` key `mls_api.base_url` (falls back to a default mock feed)
and is persisted back to settings. This proves live-feed ingestion, not just Excel.

### SMTP (outbound email)
`smtplib` sends batch outreach (`POST /api/email-queue/send-batch`) using settings
resolved from `admin_app_settings` via `_smtp_cfg` (`smtp.host`, `smtp.port`,
`smtp.username`, `smtp.password`, `smtp.sender_email`, `smtp.sender_name`,
`smtp.signature`). If SMTP isn't configured the send refuses rather than silently
succeeding. Every email gets a STOP-reply unsubscribe footer and URL linkification.
Test endpoint: `POST /api/admin/smtp/test`.

### IMAP (Sent copy + inbox poller)
`imaplib` is used two ways:

- After each send, the message is best-effort APPENDed to the IMAP Sent folder
  (`imap_host`/`imap_port`; defaults `secure.emailsrvr.com:993`; tries `Sent`,
  `Sent Items`, `INBOX.Sent`). An IMAP failure never fails a delivered send.
- The background unsubscribe/bounce poller (`_poll_unsubscribe_inbox`, looped from
  `main.py`) scans the inbox for STOP/unsubscribe replies and hard bounces,
  suppressing those agents. It runs per-territory, scoped via the RLS session vars,
  and is on by default in prod only. Also a manual button: `POST /api/admin/unsubscribe/check`.
  Application intake reads the inbox via `/api/applications/scan/read-inbox`.
  Test endpoint: `POST /api/admin/imap/test`.

### Anthropic API
Used only by the **Update Website** tool (`update_website.py`, prefix
`/api/update-website`), model `claude-sonnet-4-6`. The server proxies the
FAQ-rewrite call so `ANTHROPIC_API_KEY` (backend env) never reaches the browser.
Returns clear errors if the key is unset or the `anthropic` package is missing.
Editable answer principles persist to `data/rules_faq.json` (one locked principle
always enforced).

## Main API endpoint groups

All under the `/api` prefix, in `backend/app/api.py` (plus the separate
`update_website.py` router).

| Group | Endpoints (representative) |
|---|---|
| Auth / env | `/auth/login`, `/auth/me`, `/environment` |
| Dashboard / engagement | `/dashboard`, `/engagement/summary`, `/engagement/list` |
| Agents & matching | `/agents/match-summary`, `/agents/matches`, `/agents/matches/decide`, `/agents/matches/run`, `/agents/matches/export`, `/agents/matching-stats`, `/agents/unmatched` (+ `/make-golden`, `/make-golden-bulk`, `/link`, `/skip`), `/master-list/search` |
| Email queue | `/email-queue/filter-options`, `/buckets`, `/dtc-buckets`, `/agents`, `/templates`, `/stats`, `/send-batch`, `/preview`, `/_cleanup_test`; `/sent-history` |
| Properties | `/properties`, `/properties/load-stats`, `/properties/import` (Excel), `/properties/import-api` (RESO) |
| Call tracker | `/call-tracker` (+ `/stats`, `PUT /{cid}`), `/cold-call-matches` (+ `/stats`, `/import`, `/{mid}/decision`, `/{mid}/readd`) |
| DB query console | `/db-query/run`, `/db-query/tables`, `/row-counts` |
| Admin — settings | `/admin/settings` (GET/PUT/bulk), `/admin/encryption-status`, `/admin/reference/{kind}`, `/admin/templates` (+ history, restore), `/admin/national-report` |
| Admin — onboarding/logins | `/admin/onboard/source`, `/admin/onboard/copy`, `/admin/logins`, `/admin/logins/reset-password` |
| Admin — email infra | `/admin/smtp/test`, `/admin/imap/test`, `/admin/send-utility/agents`, `/admin/send-utility/preview` |
| Admin — master list / backup / tests | `/admin/master-list` (+ stats, import), `/admin/backup/list\|run\|download\|restore`, `/admin/test-suite/list\|run\|report` |
| Unsubscribe / bounces | `/unsubscribe` (+ `/{uid}/readd`), `/bounces`, `/admin/unsubscribe/stats\|check\|readd` |
| Applications (intake) | `/applications` (+ `/{app_id}`, `/verdict`, `/recompute`), `/applications/scan/read-inbox\|preview\|parse-run` |
| Update Website | `/api/update-website/rules` + LLM proxy |

Authentication: every `/api/*` route requires a valid bearer token except the
login-gate allowlist (`/auth/login`, `/environment`, `/auth/me`). See
[Architecture → Authentication](architecture.md#authentication).
