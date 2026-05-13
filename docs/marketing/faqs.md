# Frequently Asked Questions

## Setup

### How do I install CE FranchiseOwner on a new machine?

Run the Inno installer (`CE-FranchiseOwner-Setup.exe`). It drops the
app, the bundled MariaDB binaries, and a complete `.env` into the
install folder. First launch auto-initialises the local MariaDB
data directory and creates every table. No external database to
configure.

### What credentials do I need before first run?

Three things, all entered in the **Admin** screen:

1. **PropStream** username and password — for the daily scrape.
2. **Rackspace SMTP** username and password — for outgoing email.
   The same credentials are reused for IMAP (unsubscribe poller +
   application inbox).
3. **Anthropic API key** *(only if you'll use Pre-Underwriting)* —
   for the Q4 / Q8 PDF scans. The app degrades gracefully without
   it: those two answers simply return "API key not configured."

Everything is base-64-encoded at rest (`app_settings` table), plus
applicant PII is double-protected by per-row Fernet encryption.

## Daily use

### Where does the app find new listings?

Through PropStream saved searches you've configured under
**Admin → Saved Searches**, one row per county. The Daily Job loops
them in `sort_order`. Selenium drives a persistent Chrome user
profile so you don't get re-prompted to log in.

### How often should I run the daily scrape job?

Once per business day. Press **▶ Start Job** in *Run Daily Job*
with a date range that covers since-last-run; you can also use the
*Last Week* / *Last 3 Days* presets.

### How do I know if an email actually sent?

Every successful send writes one row to `email_log` with `sent_at`
set to `NOW()` and `is_test=0`. The **Sent History** screen reads
from that table. The *Test Runs* tab there shows your preview
sends (`is_test=1`).

## Data

### How are agent matches verified?

Auto matches are produced by `rapidfuzz` token-sort comparison
against the master list (default threshold 85%). The **Review
Matches** screen lets you bulk-approve high-confidence buckets
(100%, 90-99%) and review the others one row at a time, or export
the whole 80-89% band to Excel, mark Y/N on every row, and re-import.

Verified matches are marked `is_verified='Y' is_golden='Y'` —
that's the canonical "this listing-name maps to this master agent"
record.

### What happens to an agent who replies "unsubscribe"?

The IMAP unsubscribe poller (`email_engine/unsubscribe_poller.py`)
runs twice a day (12:00 and 16:30 local). It scans the inbox for
keywords (`stop`, `unsubscribe`, `remove me`, `opt out`,
`do not contact`, `do not email`), parses the originating agent
from the email thread, and inserts a row into `unsubscribe_list`.
Future drip sends skip any agent whose email appears in that table.

### Can I restore the database from a backup?

Yes. Admin → Backup & Restore → click *Restore* next to a backup.
You'll get a *Restore this backup?* confirmation that warns
"this will overwrite ALL current data." After restore the app
auto-relaunches via `os.execv`.

## Troubleshooting

### The app says "Chrome profile not found." What now?

The PropStream scraper needs a real Chrome user-data directory that
already has a logged-in PropStream session. Run:

```powershell
"C:\Program Files\Google\Chrome\Application\chrome.exe" --user-data-dir=C:\ChromeCEProfile
```

Sign into PropStream in that window, close it, then paste
`C:\ChromeCEProfile` into **Admin → PropStream → Chrome profile
path** and save.

### My batch email send stopped midway. Did the rest go out?

The Email Queue screen now writes a row to `system_logs` with
`module='batch_progress'` after every send. The Email Queue's
top-of-screen banner (orange while RUNNING, green for 10 min after
COMPLETE) survives navigation — if you closed the window mid-batch,
re-open the Email Queue and the banner tells you exactly how far
the worker got and whether it finished.

### Why does the Pre-Underwriting screen say "API key not configured"?

The two LLM scans (Q4 contingencies, Q8 pre-qual) need an Anthropic
API key in `app_settings.anthropic_api_key`. Without one the rest of
the gate check still runs — the affected questions just return
`NO_API_KEY` and the verdict reflects the questions that *did* run.
