# QA & Testing

## Running the test suite

```powershell
# From the project root, with the venv activated
venv\Scripts\activate
pytest
```

`pytest` and `ruff` are both declared in `requirements.txt`.

## Linting

```powershell
ruff check .
```

## Current state of the suite

The `tests/` package is scaffolded (`__init__.py` present) but does
not yet have implemented test modules in this revision. Two
concrete priorities for the next sprint:

1. **Gate-check scoring (pure logic, no I/O).** `core/gate_check_engine.py`
   exports a stable public API (`calculate_q1`,
   `calculate_q3`–`q12`, `calculate_total`). Every function is a
   pure transformation over primitive inputs — perfect for table-
   driven tests. Includes the knockout threshold logic
   (`KNOCKOUT_THRESHOLD = -50`, `KNOCKOUT_VALUE = -100`) which is
   critical to get right.
2. **Email parser regression.** `core/email_parser.py` parses
   Gravity-Forms application HTML into a flat dict mapped to
   `ce_application_dim`. The `LABEL_TO_COLUMN` dict has dozens of
   tolerant string variants. A single fixture-driven test with a
   real-shape application email locks in the mapping.

## Manual QA checklist (per release)

Until the suite catches up, run this checklist by hand before
shipping a build:

### Startup
- [ ] Source mode + `APP_ENV=dev` → app launches cleanly.
- [ ] Source mode + `APP_ENV=prod` → refuses to start with the
      "REFUSING TO START" banner unless
      `CE_ALLOW_PROD_FROM_SOURCE=1` is set.
- [ ] Frozen `.exe` + `APP_ENV=prod` → launches; audit row
      appears in `system_logs` with `module='startup'`.
- [ ] Auto-backup runs on prod startup if last backup older than
      24 h.

### Lead Engine
- [ ] **Run Daily Job** scrapes every saved search and writes new
      `raw_scrapes` rows.
- [ ] **Email Queue → AUC bucket** filters update agent counts
      live; **Search agent** narrows the list.
- [ ] **Send test email** delivers; **Start Batch** unlocks only
      after a test email succeeds.
- [ ] Batch send writes one row per send to `email_log` and a
      `batch_progress` row to `system_logs` after each send.
- [ ] Navigating away from the Email Queue mid-batch and back
      restores the orange in-progress banner; the green
      "complete" banner shows for 10 minutes after the COMPLETE
      row is written.
- [ ] Review Matches → Excel round-trip: download → mark Y/N →
      upload → counters update. Cross-band IDs in the upload
      are rejected by the band-locked UPDATE.

### Pre-Underwriting
- [ ] **Step 1 — Read inbox** pulls a new application, encrypts
      PII, inserts `ce_application_dim` row.
- [ ] **Step 2 — Run extractions** classifies each attached PDF,
      writes Q4 and Q8 results to `ce_application_api_calls`.
- [ ] 12-question grid shows scores; knockout questions are
      flagged; verdict is computed.
- [ ] Override on any single question recalculates the verdict.

### Admin
- [ ] Every tab opens without errors.
- [ ] `?` help button on every tab opens the correct doc page.
- [ ] **Docs Engine → Copy Prompt** writes to clipboard; the
      tooltip "Prompt copied" appears.
- [ ] **Backup & Restore → Run Backup Now** produces a fresh
      `.zip` in `Documents\CommissionExpress\Backups\` and rows
      appear in the *Backup history* table.
- [ ] **DB Migrations** (dev only) compares dev vs prod schema
      and the *Apply all to PROD* button runs without errors on
      an in-sync database.

### Tear-down
- [ ] App exit calls `mysqladmin shutdown` cleanly (no orphan
      `mysqld.exe` left running).
