# Deployment Guide

## Build the Windows executable

```powershell
# From the project root, with the venv activated
venv\Scripts\activate
pip install -r requirements.txt
pyinstaller CE-FranchiseOwner.spec
```

The build output lands in `dist/CE-FranchiseOwner/`. The folder
contains the `.exe`, all bundled DLLs, and the `data/`, `assets/`,
and `logs/` subfolders.

The separate Inno Setup installer (not in this repo) wraps that
folder, generates a unique install location, and drops a complete
`.env` next to the `.exe`. Distribute the installer, not the raw
`dist/` folder.

## MariaDB setup

The app ships with an embedded MariaDB. No external install needed.

1. `data/mariadb/bin/` contains `mysqld.exe`, `mysql.exe`,
   `mysqldump.exe`, `mysql_install_db.exe`. These are committed
   inside the source tree (large binaries — make sure Git LFS or a
   `.gitattributes` setting is configured if you fork the repo).
2. **First launch.** `core.db_server.start_mariadb()` checks
   whether `data/mariadb_data/mysql/` exists; if not, it runs
   `mysql_install_db.exe --datadir=...` to create the system tables.
3. **Every launch.** `mysqld.exe --port=3307 --bind-address=127.0.0.1
   --skip-networking=OFF` is spawned as a background process. The
   `Popen` handle is kept alive for the lifetime of the app; an
   `atexit` hook calls `mysqladmin.exe shutdown` to stop it cleanly
   on exit.
4. **Schema.** `core.database.init_database()` runs
   `CREATE TABLE IF NOT EXISTS` for every table plus the
   `_migrate_*` helpers. Safe to call on every boot.
5. **Auto-backup.** On a production startup, if no
   `module='backup_manager'` row exists in `system_logs` within the
   last 24 hours, a daemon thread runs a `mysqldump` of the live DB
   into the user's `Documents\CommissionExpress\Backups\` folder and
   prunes anything beyond the 30-most-recent.

## Required `.env` variables

The installer writes a complete `.env` next to the `.exe`. For source
runs, place a `.env` next to `config.py`. Variables:

| Variable | Default (dev) | Required in prod |
|---|---|---|
| `APP_ENV` | `dev` | **`prod`** — guards the "refuse to start from source if prod" check |
| `DB_HOST` | `127.0.0.1` | Same — DB is always local |
| `DB_PORT` | `3307` | Same — embedded MariaDB |
| `DB_USER` | `root` | Whatever the installer set |
| `DB_PASSWORD` | empty | Whatever the installer set; empty is acceptable for an unexposed localhost server |
| `DB_NAME` | `ce_franchiseowner_dev` | **`ce_franchiseowner`** |
| `CE_ALLOW_PROD_FROM_SOURCE` | unset | Set to `1` only for emergency maintenance scripts |

## Promoting dev → prod (one-shot setup)

The `tools/setup_prod_db.py` script copies the dev schema and seed
data into a fresh `ce_franchiseowner` database. Run from the project
root:

```powershell
python tools/setup_prod_db.py
```

Type `YES` when prompted. It is idempotent — re-running it after
the first successful run is a no-op for seed data (it only copies
seed rows into empty tables).

## Resetting the dev database

```powershell
python tools/reset_dev_db.py
```

Truncates every transaction table in dependency order
(`FOREIGN_KEY_CHECKS=0` is flipped briefly as belt-and-braces).
Leaves dim tables, `app_settings`, `email_templates`, and
`master_list` alone, so the next launch isn't a blank slate.

## Distributing the installer

1. Build the `.exe` (above).
2. Run the Inno Setup script (lives in a sibling repo).
3. Ship the resulting `CE-FranchiseOwner-Setup.exe` to the operator.
4. Provide the operator with three credential bundles:
   - PropStream username + password
   - Rackspace SMTP / IMAP username + password
   - Anthropic API key (optional)

On first launch the operator enters those credentials in Admin, the
app saves them base-64-encoded into `app_settings`, and from then on
the app is self-contained.

## Logs

| Path | Written by |
|---|---|
| `logs/` *(app data dir)* | General app logs |
| `data/logs/email_sends.log` | Flat-file mirror of every email send for offline audit |
| `logs/doc_run.log` | Docs Engine run history — read by the *Docs Engine* admin tab to colour its status banner |
| `system_logs` table | All audit events: `startup`, `db_warn`, `backup_manager`, `batch_progress`, `migration`, screen-specific changes |
