# Feature Guides

This page documents the three products inside CE FranchiseOwner.
Each section covers what the product does, how to use it day-to-day,
and which settings matter. Written for the franchise owner using
the app, not the developer maintaining it.

---

## Lead Engine

### What it does

The Lead Engine finds real-estate agents whose listings just went
Active Under Contract and emails them a personalised drip
sequence. It runs in four stages:

1. **Scrape** — Selenium drives Chrome against PropStream using
   your saved searches (one per county). Login is automatic via a
   persistent Chrome profile, with human-paced typing if the
   session has expired.
2. **Load** — scraped rows land in the `raw_scrapes` table, then
   are normalised into `property_loads` with one row per listing.
3. **Match** — `rapidfuzz` compares each listing's
   `mls_agent_name` against your `master_list` of licensed SC
   agents. Auto-matches over 85% confidence are written to
   `agent_match_with_master` with `match_status='PENDING'`.
   High-confidence matches (100% and 90-99%) can be bulk-approved;
   the 80-89% band is reviewed through an Excel round-trip
   (download, mark Y/N, upload).
4. **Send** — the Email Queue groups verified agents by number of
   active listings (the AUC bucket) and days to settlement, lets
   you pick a template and send a test to yourself, and then
   sends to the whole batch through Rackspace SMTP. Every send
   writes one row to `email_log`.

### How to use it

From the landing page click **Lead Engine** to open the main
window with its sidebar.

1. **Run Daily Job** — pick a *From* and *To* date (or use a
   preset: Last Week / Last 3 Days / Last 7 Days / This Month).
   Click **Start Job**. The right panel shows live activity per
   county as the scrape runs.
2. **Initial Load → Property Data** — one-time import of an
   exported PropStream CSV per county. The pre-checks phase
   validates that every required column is present before
   anything is inserted.
3. **Initial Load → Agent Matching → Review Matches** — the
   Excel round-trip: click **Download Review File** to export a
   3-sheet workbook (100%, 90-99%, 80-89%). The first two are
   pre-filled with `Y` (Same); the 80-89% sheet is empty. Type
   `Y` or `N` next to every row, save, and click **Upload &
   Apply Decisions**. The upload runs two batched UPDATE
   statements (one for approves, one for rejects) and refreshes
   the top counter bar.
4. **Email Queue** — pick an AUC bucket on the left rail, then
   a Days to Close range, then a template. Use the **Search
   agent** box to filter by name. Click **Send test email** to
   your own address. After a successful test, **Start Batch**
   unlocks.
5. **Sent History** — chronological view of every send from
   `email_log`. Filter by status, by template, or by template
   version.

### Key settings

Configured in **Admin**:

- **Email / SMTP** — sender name, sender email, SMTP host
  (`secure.emailsrvr.com`), port (`587`), username, password,
  signature. Save is gated on a successful test email.
- **Templates** — 10 single-AUC templates plus 10 multi-AUC
  templates. Each save snapshots a version row in
  `email_template_history`.
- **Saved Searches** — one row per PropStream saved-search name
  with a `sort_order` for the loop.
- **Master List** — Excel import of the SC agent master list
  (with phone). Old rows soft-deleted; new rows get a new
  `load_batch_id`.
- **Unsubscribe** — view the suppression list and re-add removed
  agents. The list is populated automatically by the IMAP
  unsubscribe poller (12:00 and 16:30 local each day).

---

## Call Tracker

### What it does

Manages the cold-call follow-up workflow for agents you've
previously contacted but who haven't converted. The Call Tracker
is a separate top-level window with four tabs.

**Agent Matching for Calling** imports a "Previous Clients"
Excel file (one row per past deal), counts deals per agent, and
fuzzy-matches each unique agent against your master list using
the same rapidfuzz engine the Lead Engine uses. Results are
staged in `cold_call_matches` with `match_status='PENDING'` for
human review.

**Review Call Matches** is the human-review queue. Pending,
Approved, and Rejected tabs paginated 50 rows at a time. Per-row
or bulk Approve and Reject. On approve, the agent's phone (from
the master list where available, falling back to the phone in
the imported file) is written to the `cold_call_log` table —
either inserting a new row or updating the existing one by
agent name.

**Call Log** is the daily call worksheet. One row per approved
agent. Each row tracks:

- `call_status` — Not called by default; flips to Called once
  the franchise owner clicks the row's call button.
- `outcome` — picked from a dropdown: Interested, Converted,
  Call me back, Left voicemail, Not interested, Do not call.
  Each outcome paints the row a different soft colour so the
  call list reads at a glance.
- `notes` — free-form text.
- `followup_date` — sets a callback date; overdue follow-ups
  are highlighted.

**Add / Re-add** lists agents whose match was Rejected
(parked) so you can put them back into the call list when one
of them reaches out unprompted. Clicking *Re-add* prompts for
a short reason (audit trail) via a one-field dialog with
default text "Agent reached out", then flips the
`match_status` back to `APPROVED`, stamps `readded_by`,
`readded_at`, and `readded_reason`, and re-inserts the agent
into `cold_call_log`.

### How to use it

From the landing page click **Call Tracker** to open its
window.

1. **Agent Matching for Calling** — click **Import Previous
   Clients Excel** and pick the `.xlsx`. The matcher runs
   and you'll see counts of new pending matches.
2. **Review Call Matches** — open the Pending tab. For each
   match either click the green check (Approve) or the red X
   (Reject). Or use the bulk **Approve All Selected** /
   **Reject All Selected** buttons after ticking the
   checkboxes.
3. **Call Log** — work the list top to bottom. Pick up the
   phone for each agent, change the outcome dropdown to the
   appropriate value, type notes, set a follow-up date if
   needed.
4. **Add / Re-add** — review the parked list periodically.
   When an agent calls you, find them here and click **Re-add
   →**, type a reason (or accept the default "Agent reached
   out"), confirm. They drop straight back into the Call Log
   with `call_status='Not called'`.

### Key settings

The Call Tracker has no Admin-tab settings of its own. Its
data sources are:

- **`master_list`** — populated through Admin → Master List.
  Drives the fuzzy match and supplies the phone number used
  on approval.
- **`cold_call_matches`** — staged matches awaiting review.
  Approve/Reject/Re-add audit columns
  (`reviewed_by`, `reviewed_at`, `readded_by`, `readded_at`,
  `readded_reason`) are kept indefinitely.
- **`cold_call_log`** — the active call list. Unique key on
  `agent_name`; duplicate imports update in place rather than
  creating new rows.

---

## Pre-Underwriting / Gate Check

### What it does

Reads completed Commission Express application emails from a
dedicated IMAP mailbox, parses each one into encrypted
database rows, classifies and scans the attached PDFs, and
runs every application through a 12-question gate check that
returns a fundability verdict.

**Email parsing.** The application form (built on Gravity
Forms) emits a long HTML email when an agent submits. The
parser in `core/email_parser.py` walks the email body
section-by-section (Agent Information, Company Information,
Property Contract Identification, Settlement / Title or
Escrow Company Information, Lender Information, Contract
Information, Receivable, Other Agent Information, PDF Files)
and maps each labelled field to a column in
`ce_application_dim`. Every PII column (SSN, home address,
bank routing/account, driver's licence, contact numbers) is
Fernet-encrypted before insertion using the key in
`app_settings.pii_encryption_key`.

**PDF classification.** Each `gf-download` URL inside the
application email is parsed by `core/pdf_extractor.py` and
classified as one of: `mls`, `purchase`, `prequal`,
`compensation`, `license`, `misc`, or `unknown`. License PDFs
are never downloaded.

**LLM scans.** Two questions use the Anthropic API
(`claude-haiku-4-5` with the PDFs beta header) against the
attached document:

- **Q4** — open contingencies in the purchase contract
- **Q8** — pre-approval / proof of funds

Every API call is logged to `ce_application_api_calls` with
tokens in/out and cost in USD. Without an API key configured,
both questions return `NO_API_KEY` and the rest of the
gate check still runs.

**Gate check scoring.** The remaining ten questions are
scored locally by `core/gate_check_engine.py` (deal history,
under-contract status, bankruptcy, UCC-1, down-payment ratio,
deal type, pipeline pending, active listings, prior-year
income). Any score `<= -50` is treated as a **knockout** —
the application is auto-flagged for rejection regardless of
the other answers.

### How to use it

Open the Pre-Underwriting window from the main app.

1. **New Applications** tab — Step 1 *Read inbox* pulls
   unread messages from the application mailbox, parses each
   one, encrypts PII, and inserts a row into
   `ce_application_dim`. New rows appear in the table on
   the right.
2. **New Applications** tab — Step 2 *Run extractions*
   classifies each attached PDF and fires the Q4 and Q8 LLM
   scans. The 4-column inline grid (Q1-Q12) populates with
   the score per question plus a verdict.
3. Override any individual answer by clicking its cell and
   typing a replacement. The verdict recalculates live.
4. Click *Save verdict* to lock the application's status.
   The row moves to the **Reviewed** tab; the Dashboard tab
   updates its KPI strip.

### Key settings

Configured in **Admin**:

- **`app_settings.imap.application_*`** — the dedicated
  mailbox credentials (typically a sub-folder of the
  Rackspace SMTP account).
- **`app_settings.anthropic_api_key`** — Claude API key
  (base-64 at rest). Optional; without it the LLM-driven
  questions skip cleanly.
- **`pii_encryption_key`** — Fernet key in `app_settings`.
  Auto-generated on first run; never expose this value.
- **Reference Data** admin tab — the dim tables for MLS
  statuses, prequal terms, and pre-approval terms used by the
  local field parsers in `core/field_parsers/`.
