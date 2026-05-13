# Deployment Guide

!!! info "Placeholder"
    Run the **Docs Engine** tab to regenerate this page from the
    current state of the PyInstaller spec and the installer scripts.

## Build the Windows executable

```powershell
# From project root, with the venv activated
pyinstaller CE-FranchiseOwner.spec
```

The bundled `.exe` lands in `dist/CE-FranchiseOwner/`. The Inno
installer (separate repo) wraps this folder and drops a complete
`.env` next to the `.exe` at install time.

## MariaDB setup

The app ships with an embedded MariaDB. On first launch
`core/db_server.py` runs `mysql_install_db` against
`data/mariadb_data/`, then starts `mysqld.exe` on `127.0.0.1:3307`.
Schema is created by `core.database.init_database()` (idempotent
`CREATE TABLE IF NOT EXISTS`).

## Required `.env` variables

| Variable | Purpose |
|---|---|
| `APP_ENV` | `dev` (default) or `prod` |
| `DB_HOST` | Default `127.0.0.1` |
| `DB_PORT` | Default `3307` |
| `DB_USER` | Default `root` |
| `DB_PASSWORD` | Empty string for local embedded server |
| `DB_NAME` | `ce_franchiseowner` (prod) / `ce_franchiseowner_dev` (dev) |

_To be expanded._
