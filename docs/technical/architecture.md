# Architecture

## Folder structure

```text
CE-FranchiseOwner/
├── main.py                # App entry point. Guards prod-from-source.
├── config.py              # Loads .env, exposes APP_ENV + ACTIVE_DB.
├── requirements.txt
├── CE-FranchiseOwner.spec # PyInstaller spec (Windows .exe build).
│
├── core/                  # Business logic, DB, lifecycle
│   ├── database.py        # init_database() + per-table migrations
│   ├── db_server.py       # Bundled MariaDB lifecycle + auto-backup
│   ├── settings_manager.py# app_settings get/save (with PII encoding)
│   ├── backup_manager.py  # mysqldump + zip + restore + rotation
│   ├── crypto.py          # Fernet PII encryption helpers
│   ├── email_parser.py    # Application-email HTML → ce_application_dim
│   ├── pdf_extractor.py   # PDF classify + fetch + pdfplumber text
│   ├── llm_scanner.py     # Anthropic API client (Q4, Q8 scans)
│   ├── gate_check_engine.py # 12-question scoring + knockout logic
│   └── field_parsers/     # Per-question heuristic extractors
│
├── gui/                   # Tkinter UI
│   ├── splash.py
│   ├── login_window.py
│   ├── landing_page.py    # Lead Engine / Call Tracker chooser
│   ├── main_window.py     # Lead Engine main window + sidebar
│   ├── admin_window.py    # Thin wrapper that mounts AdminScreen
│   ├── call_tracker_window.py
│   ├── docs_tab.py        # Admin → Docs Engine tab
│   ├── components/        # Sidebar, Header, dev banner
│   └── screens/
│       ├── admin_screen.py            # 5200-line tabbed Admin
│       ├── email_queue_screen.py      # AUC bucket picker → batch send
│       ├── review_matches_screen.py   # Excel round-trip review
│       ├── run_daily_job_screen.py    # Daily scrape trigger
│       ├── sent_history_screen.py     # email_log viewer
│       ├── upload_history.py          # Property import landing
│       ├── pre_underwriting_screen.py # Application gate check
│       ├── cold_call_tab.py
│       ├── cold_call_matching_screen.py
│       ├── review_cold_call_matches_screen.py
│       ├── agent_matching_screen.py
│       ├── manual_match_screen.py
│       └── db_query_screen.py
│
├── scraper/               # PropStream scraping
│   ├── propstream_scraper.py            # Selenium (current)
│   ├── propstream_scraper_playwright.py # Playwright (experimental)
│   └── agent_matcher.py                 # rapidfuzz matching
│
├── email_engine/          # Email send + IMAP poll
│   ├── email_sender.py    # Single SMTP send entry point
│   └── unsubscribe_poller.py # IMAP poll, 12:00 and 16:30 local
│
├── utils/                 # Cross-cutting helpers
│   └── help_links.py      # Docs URL registry + open_help()
│
├── tools/                 # One-off operator scripts
│   ├── setup_prod_db.py   # Copy schema/seeds DEV → PROD
│   └── reset_dev_db.py    # Truncate transaction tables in DEV
│
├── tests/                 # pytest suite (currently scaffolded)
├── assets/                # Icons, logos
├── data/                  # Bundled MariaDB + master-list xlsx
└── logs/                  # App + scrape + email + doc-run logs
```

## Tech stack

| Layer | Technology |
|---|---|
| Language | Python 3.14 |
| GUI | tkinter / ttk (PySide6 migration planned for large tables) |
| Browser automation | Selenium with a persistent Chrome user profile, plus a Playwright variant under `scraper/propstream_scraper_playwright.py` |
| Database | MariaDB 11.x embedded; the `mysqld.exe` is shipped inside `data/mariadb/bin/` |
| Email | `smtplib` (outgoing) + `imaplib` (inbox polling), both over Rackspace |
| Fuzzy matching | `rapidfuzz.fuzz.token_sort_ratio` |
| Excel I/O | `openpyxl` + `pandas` |
| PII encryption | `cryptography.Fernet` (AES-128-CBC + HMAC-SHA256) |
| PDF extraction | `pdfplumber` for local text, Anthropic API for LLM scans |
| LLM | `claude-haiku-4-5` via the Anthropic Messages API, with the `pdfs-2024-09-25` beta header to support PDF document blocks |
| Packaging | PyInstaller (Windows .exe via `CE-FranchiseOwner.spec`) |

## Application startup flow

1. `python main.py` (or the frozen `.exe`) calls `start()`.
2. **Safety guard.** If running from source AND `.env` says
   `APP_ENV=prod`, refuse to launch unless
   `CE_ALLOW_PROD_FROM_SOURCE=1` is set. This prevents an accidental
   dev session from writing to the production database.
3. **Splash + MariaDB.** A `SplashScreen` shows "Starting database…".
   `core.db_server.start_mariadb()` spawns the bundled `mysqld.exe`
   on `127.0.0.1:3307`, initialising
   `data/mariadb_data/` on first run.
4. **Schema.** `core.database.init_database()` runs every `CREATE
   TABLE IF NOT EXISTS` plus migration helpers (`_migrate_*`). Safe
   to call on every boot.
5. **Audit row.** A `startup` row is inserted into `system_logs` with
   `ENV` and `DATABASE()` so we can always answer "which DB was this
   process pointed at when X was sent?"
6. **Auto-backup.** If `APP_ENV=prod` and no `backup_manager` row
   exists in `system_logs` from the last 24 hours, a daemon thread
   runs a `mysqldump` of the live DB into the user's
   `Documents\CommissionExpress\Backups\` folder.
7. **Unsubscribe scheduler.** A daemon thread schedules
   `UnsubscribePoller.run_check_thread()` at 12:00 and 16:30 local.
8. **Login → Landing.** `LoginWindow` accepts a username/password;
   on success, `LandingPage` lets the user pick Lead Engine or Call
   Tracker. (License key activation is wired up but disabled for
   demo — re-enable by wrapping `_run_login()` in `LicenseWindow`.)

## Key design decisions

- **Embedded database.** MariaDB ships inside the install folder.
  No external DB to provision, no PII leaving the machine. Pairs
  with Fernet at-rest encryption for double-defence on applicant
  personal data.
- **`curr_rec_ind` soft-delete pattern.** Used on `property_loads`,
  `master_list`, `agent_match_with_master`, and the
  `ce_application_*` tables. Rows are never `DELETE`'d; the
  historical row is set to `'N'` and a new `'Y'` row is inserted.
  Pairs with `load_batch_id` for time-series queryability.
- **`app_settings` as KV store.** Single `(setting_key, setting_value)`
  table holds every configurable value. Sensitive keys (matching
  `password`, `secret`, `token`, `api_key`) are base-64-encoded at
  rest. The DB Query screen masks them on display.
- **`system_logs` as the audit log.** App startup, DB warnings,
  reference-data CRUD, migration applies, backup runs, batch sends,
  and doc-engine runs all funnel here. Querying by `module` filter
  retrieves the relevant subset.
- **Two-window navigation.** The `LandingPage` after login chooses
  between `MainWindow` (Lead Engine) and `CallTrackerWindow`. Each
  top-level window has its own header and sidebar so context never
  spills across products.
- **GUI ↔ business logic isolation.** Every screen calls into the
  `core/`, `scraper/`, and `email_engine/` modules; none of the
  business logic imports tkinter. A future Qt rewrite swaps only
  the `gui/` layer.
