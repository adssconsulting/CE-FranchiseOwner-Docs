# Architecture

!!! info "Placeholder"
    Run the **Docs Engine** tab to regenerate this page from the
    current codebase.

## Folder structure

```text
CE-FranchiseOwner/
├── core/              # App lifecycle, license, auth, DB, settings
├── gui/               # tkinter windows, components, screens
├── scraper/           # Selenium / PropStream + agent matching
├── email_engine/      # Drip sequence + SMTP sending
├── utils/             # Shared helpers
├── assets/            # Icons, images
├── data/              # Embedded MariaDB
├── logs/              # App + scrape logs
└── tests/             # pytest suite
```

## Tech stack

- GUI: Python + tkinter (PySide6 migration planned)
- Browser automation: Selenium + persistent Chrome profile
- Database: MariaDB embedded
- Email: smtplib via Rackspace SMTP
- Fuzzy matching: rapidfuzz
- Excel I/O: pandas + openpyxl
- Packaging: PyInstaller (Windows .exe)

## Application startup

_To be detailed._

## Key design decisions

_To be detailed._
