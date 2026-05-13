# Feature Guides

Each section below describes one feature: what it does, how to use
it, and which settings matter. Written for the franchise owner who's
using the app, not building it.

---

## Lead Engine

**What it does.** Loops through your saved PropStream searches,
extracts every Active Under Contract listing across the date range
you pick, matches each listing's agent against your master agent
list, and queues the matched agents for a personalised email send.

**How to use it.**

1. **Home → Lead Engine** to open the main window.
2. **Run Daily Job** (sidebar): pick a *From* date and a *To* date,
   or click a preset (Last Week / Last 3 Days / Last 7 Days / This
   Month). Click **Start Job**. The right panel shows live activity
   per county as the scrape runs.
3. **Email Queue** (sidebar): once data is loaded, pick an
   **AUC bucket** (the agent's number of currently-active listings
   on the left rail), then a **Days to Close** range, then review
   the agents that appear. Type in the **Search agent** box on the
   table to filter by name. Run **Send test email** to a single
   address before unlocking the **Start Batch** button.

**Key settings.**

- `app_settings.smtp.*` — outgoing email host, port, sender name and
  email, base-64-encoded password, signature.
- `saved_searches` table — one row per PropStream saved search; loop
  order controlled by `sort_order`.
- `master_list` — the SC agent roster used for fuzzy matching.

---

## Call Tracker

**What it does.** Separate top-level screen for the cold-call follow-
up workflow. Tabs are: *Agent Matching for Calling*, *Review Call
Matches*, *Call Log*, and *Add / Re-add*.

- **Agent Matching for Calling** imports a list of agents-to-call
  (typically pulled from MLS exports), fuzzy-matches them against
  the master list, and stages results.
- **Review Call Matches** lets the owner approve or reject each
  staged match. Approved agents land in the call list.
- **Call Log** is the daily call worksheet — name, phone, status
  (Not called / Called / Voicemail / Bad number / Not interested /
  Interested / Closed), notes, follow-up date.
- **Add / Re-add** brings parked (rejected) agents back into the
  call list when one of them reaches out unprompted. Requires a
  short reason for the audit trail.

**Key settings.** None. Driven entirely by the staged data.

---

## Gate Check (Pre-Underwriting)

**What it does.** Reads completed CE application emails out of a
dedicated IMAP mailbox, parses the HTML body into structured DB
fields, encrypts every PII column, classifies and scans each
attached PDF, and runs the 12-question gate check to produce a
verdict.

**12 questions, knockout-style.** Q1 (deal history), Q3
(under-contract status), Q4 (open contingencies on the purchase
contract), Q5 (bankruptcy), Q6 (UCC-1), Q7 (down-payment ratio),
Q8 (pre-approval / proof of funds), Q9 (deal type), Q10–12
(pipeline, listings, income). Any score ≤ –50 is a *knockout* — the
application is auto-flagged "Reject — knockout."

**How to use it.**

1. Open **Pre-Underwriting** from the main window.
2. **New Applications** tab — Step 1 pulls fresh emails from the
   inbox and inserts parsed rows. Step 2 runs the LLM scans (Q4 and
   Q8) against the application's PDFs.
3. Inline 4-column grid shows Q1–Q12 scores plus the verdict. Click
   any row to expand and override individual answers.
4. **Reviewed** tab shows all applications where a verdict has been
   saved.

**Key settings.**

- `app_settings.imap.*` — application-email mailbox credentials.
- `app_settings.anthropic_api_key` — Claude API key. If missing,
  Q4 and Q8 return `NO_API_KEY` without making a network call.
- `pii_encryption_key` — Fernet key, auto-generated on first run.

---

## Email / SMTP

**What it does.** Centralised configuration for outgoing email.
Validates connectivity by sending a real test email before the save
button unlocks.

**How to use it.** Admin → Email / SMTP. Enter sender name, sender
address, host (`secure.emailsrvr.com`), port (`587`), username,
password, delay between sends, signature. Click **Send test email**.
After a successful test the **Save** button enables.

**Key settings.** All `app_settings.smtp.*` keys. Password is
base64-encoded at rest.

---

## Templates

**What it does.** 10 single-AUC drip templates plus 10 multi-AUC
templates (used when an agent has 2+ active listings). Each template
has a subject, a per-step opening line, and a shared body. Versions
are tracked: every save snapshots a row in `email_template_history`.

**How to use it.** Admin → Templates. Pick the template number, edit
subject/opening/body, click **Save**. The version increments
automatically.

**Key tags inside templates.**

- `[FirstName]`, `[AgentName]`, `[Office]`
- `[PropertyAddress]` (single-AUC only)
- `[PropertyTable]` (multi-AUC only — replaced with an HTML table of
  the agent's open listings at send time)
- `[DaysToClose]`, `[AvgDaysToClose]`, `[SenderName]`

---

## Saved Searches

**What it does.** One row per PropStream saved-search name, in the
order you want them looped.

**How to use it.** Admin → Saved Searches. Add a search name (must
match the saved-search name inside PropStream exactly). Set the
sort order. The Daily Job loops through them in that order.

---

## Master List

**What it does.** Import the SC real-estate agent master list from
an Excel file. Old rows are soft-deleted (`curr_rec_ind='N'`); new
rows get a fresh `load_batch_id`. Phone numbers are imported so the
cold-call workflow can use them after approval.

**How to use it.** Admin → Master List. Click *Import*, pick the
`.xlsx`, review the row count, confirm. The previous import stays
in DB history but is no longer "current."

---

## Send Utility

**What it does.** Re-send any past email from `email_log` to an
override address (your own). Used for verifying a template still
renders correctly without polluting Sent History.

**How to use it.** Admin → Send Utility. Pick an agent from the
dropdown (sourced from past sends), enter your test email, click
**Send Preview**. Subject is prefixed with `[PREVIEW]` and the row
is *not* written to `email_log`.

---

## Backup & Restore

**What it does.** One-click `mysqldump` of the entire DB into a
timestamped `.zip` under
`Documents\CommissionExpress\Backups\`. Auto-runs on production
startup if no backup has been taken in the last 24 hours. Keeps
the 30 most recent backups; older ones are pruned automatically.

**How to use it.**

- **Backup folder** — change with the *Browse* button. Saved to
  `app_settings.backup_directory`.
- **Run Backup Now** — manual button, runs in a background thread.
- **Backup history** — table of every existing backup with size and
  date; per-row *Restore* and *Delete* buttons. Restore replaces
  the running database and auto-restarts the app via `os.execv`.

---

## DB Migrations *(dev only)*

**What it does.** Compares the live schema of the development
database against production, generates the `ALTER TABLE` /
`CREATE TABLE` statements needed to bring prod up to date, and
applies them on confirmation.

**How to use it.** Only visible when `APP_ENV=dev`. Admin → DB
Migrations.

1. Click **Compare now** — populates the *Schema Differences* panel
   per-table.
2. Review the generated SQL in the dark code panel.
3. Click **Apply all to PROD** to run them — every applied
   migration is recorded in `ce_franchiseowner.db_migration_log`.

---

## Docs Engine

**What it does.** This very documentation site is regenerated by
copying a prompt from this tab into Claude Code. Claude reads the
codebase and rewrites every page in `CE-FranchiseOwner-Docs`.
GitHub Actions deploys the rebuilt site to GitHub Pages within
2 minutes of the push.

**How to use it.**

1. Admin → Docs Engine.
2. Click **📋 Copy Prompt**.
3. Switch to VS Code with the source repo open; paste the prompt
   into Claude Code.
4. Wait for Claude to finish. The Docs Engine tab will show
   *"Docs are up to date"* after the success line is appended to
   `logs/doc_run.log`.
5. **🌐 View Docs Site** opens this site.
