# Feature Guides

A plain-language tour of every screen in CE FranchiseOwner. Each section
explains **what it does**, **how to use it**, and the **key settings** you
can change. No technical knowledge required.

The app is organized as a left-hand menu of screens. We'll go through them
roughly in the order you'd use them: load your data, match agents, email
them, follow up, and manage everything from Admin.

---

## Lead Engine { #lead-engine }

The Lead Engine is the heart of the product. Every day it finds real-estate
agents whose listings just went **Active Under Contract**, matches those
agents to people you can email, and sends a personalized email sequence —
the right message, to the right agent, at the right time. The next four
screens (Dashboard, Properties, Review Matches, Email Queue) make up the
Lead Engine.

---

## Dashboard

**What it does:** The home screen. It gives you a single snapshot of your
whole operation — how many agents are in your master list, how many
property listings are loaded, how many emails you've sent, your match
results, applications received, unsubscribes, and your cold-call matches.
At the top sits a **Human Engagement Count** panel that highlights every
time a real person responded to your outreach.

**How to use it:**

- Open the page — everything loads automatically; there are no buttons to press.
- Read the row of number cards for an at-a-glance health check.
- In the Human Engagement panel, click any of the four categories —
  **Applications Received**, **Unsubscribe Requests**, **Email Replies**, or
  **Call Tracker** — to see the agents behind that number and when each engaged.

**Key settings:** None — this is a read-only overview. The guiding idea: every
engagement counts as a win, even an unsubscribe, because it means a human read
and responded.

---

## Properties

**What it does:** Where you load the property listings you'll market against.
You import PropStream county exports (the "Active Under Contract" listings),
and the page checks them, loads the new ones, and shows the current set.
New addresses are added; addresses already loaded for a county are skipped
automatically, so you never get duplicates.

**How to use it:**

- At the top, a banner shows your **Last Import**, what your **Next Import
  Should Cover** (auto-detected from the listing dates), and your **Total Runs**.
- To import from files: click **Choose Folder** (loads every spreadsheet in a
  folder) or **Choose Files** (pick one or several). One county per file.
- Click **① Run Pre-Checks** — this validates each file, counts the rows, and
  tells you which files are ready, without changing anything.
- Click **② Process Import** to load the ready files. You'll see how many new
  rows were added and how many duplicates were skipped.
- There's also an **Import from MLS API** panel for pulling listings from a
  live data feed instead of a spreadsheet — paste the feed's web address, run
  the pre-check, then import.
- Use the **search box** at the bottom to find listings by agent, address, or city.

**Key settings:** The county is figured out automatically from each file's name.
You can clear your selection and start over at any time.

---

## Review Matches

**What it does:** Connects the listing agents found in your imported properties
to actual people you can email. It works in two modes depending on whether your
territory has a master agent list. A **mode selector** at the top lets you choose
**Master Agent List Present** or **No-Master Agent List**. The app defaults to
the right one for your branch the first time, but you can switch freely.

### Mode 1 — Master Agent List Present

Matches the agents on your imported listings against your existing master list,
so you don't re-create people you already have.

- Click **Run Matching** after importing new properties. Already-matched agents
  are skipped; only brand-new agents come up for review. (Exact 100% matches are
  confirmed automatically and never shown.)
- Review results in confidence bands (high down to low). For each agent, click
  **Same** to confirm the match or **Different** to reject it. Decisions are
  permanent — a decided agent never reappears.
- Use the tabs — **For Review**, **Matched**, **Rejected**, **Not in Master** —
  to move between groups.
- For speed, tick the checkboxes and use **Select All**, then **Mark Selected —
  Same** or **Mark Selected — Different** to decide many at once.
- On the **Not in Master** tab (agents with listings but no one in your master
  list to match): click **★ Make Golden** to add a valid agent as a new emailable
  record, **🔗 Link** to attach them to someone already in your master list, or
  **Skip** to remove them. **Auto-add clean names** promotes all the
  obviously-good "First Last + email" entries in one click.
- **Export to Excel** saves the match list.

### Mode 2 — No-Master Agent List

For new markets (e.g. New Mexico, North Carolina) that have no master list to
match against. Instead of reviewing matches, you clean up the raw imported agents
directly. They're split into two buckets by their own data quality:

- **Agent Info Clean** — has a first name, last name, and a valid-format email.
  Ready to email.
- **Agent Info Email Missing** — the email is missing or malformed.

How to use it:

- In **Agent Info Clean**, use **Select All** / **Clear All** to choose agents,
  then click **✓ Make Emailable** to add them to your outreach list.
- In **Agent Info Email Missing**, click **✎ Fix & add** on a row to correct the
  email or name (which moves them into Clean), or **Skip** to remove them.

**Key settings:** Matching uses name-similarity scoring with an 85% threshold by
default (configurable in Admin → Master List). No AI is involved.

---

## Email Queue

**What it does:** The drip-email campaign. You filter your matched, emailable
agents, pick which to contact, choose the right message in the sequence,
optionally send yourself a test, and send the real batch. It tracks how many
times each agent has been contacted so the right follow-up template is chosen
automatically.

**How to use it:**

- The **left rail** lets you pick an **AUC bucket** (agents grouped by how many
  active-under-contract listings they have) and see **Days to close** breakdowns.
- Expand **Listing filters** to narrow by county, city, brokerage, listing status,
  and price range.
- Expand **Contact & repeat agent filters** to target, e.g., **new agents only**,
  **exclude already contacted**, **repeat agents only**, or **new addresses only**.
- The **📦 New since last email** toggle (on by default) limits the list to
  listings loaded *after* your last real send — the genuinely new outreach. If
  nothing is new, the page shows zero and explains why. Untick it to work your
  full current list.
- Each row shows the agent, listing count, days to close, how many times they've
  been contacted ("new" or "X× sent"), and the template that will be used. A
  **Multi** badge marks agents with several listings. Click **Preview** to see
  the exact email.
- **Cadence/drip:** the app auto-assigns each agent's template by how many times
  they've been emailed (1st Contact, 2nd Contact, …), with a separate set of
  "Multi" templates for agents with multiple listings. Override any agent's
  template with the dropdown on their row.
- **Send a test first (recommended):** type a test address and click **Send 1 Test**.
- Click **🚀 Start Batch (LIVE)** — an approval popup summarizes exactly how many
  real emails will go out. Confirm to send. A progress bar tracks the send, and a
  summary card reports how many were sent, skipped, or failed.

**Key settings:** **Batch name**, **how many to send** (25/50/100/All), and a
**gap between sends** (no delay up to 90 seconds) to pace your sending. Agents who
unsubscribed or bounced are skipped automatically; sending is blocked if email
isn't configured (with a link to set it up).

---

## Sent History

**What it does:** A searchable record of every email you've sent, grouped by agent.
Use it to confirm what went out, to whom, when, and with which template.

**How to use it:**

- Switch between the **Sent** tab (real emails) and the **Test Runs** tab.
- Filter by time period — all time, last 7, 30, or 90 days.
- Search by agent name or email.
- A summary line shows total emails, unique agents reached, and how many were
  contacted more than once.

**Key settings:** Read-only. Each row shows subject, template (and version),
batch name, send time, and status.

---

## Call Tracker { #call-tracker }

**What it does:** Manages a cold-calling campaign to previous Commission Express
clients. Import a list of past clients, match them to your master list, review
and approve those matches, then work a call list — logging outcomes, notes, and
follow-up dates — with ready-made call scripts beside you.

**How to use it (four tabs):**

- **Agent Matching for Calling:** Upload your "Previous Clients" spreadsheet (one
  row per deal). The app counts each agent's deals and matches names to your
  master list, staging results for review.
- **Review Call Matches:** Go through the staged matches and click **Approve** or
  **Reject** on each.
- **Call Log:** Your working call list of approved agents. For each, pick an
  **outcome** (Interested, Converted, Call me back, Left voicemail, Not interested,
  Do not call), and click **Notes** to add notes and a follow-up date. Stat cards
  track Total / Not Called / Interested / Do Not Call. A side panel provides
  **Call Scripts** — openers, a voicemail script, objection responses, a referral pitch.
- **Add / Re-add:** Brings rejected agents back into the call list (a reason is required).

**Key settings:** Search and filter the call log by agent, email, or outcome.
Importing a new file clears existing pending matches and stages fresh ones.

---

## Pre-Underwriting Scanner { #gate-check }

**What it does:** A triage tool that pre-screens commission-advance applications
before you spend time on them. It reads new application emails, extracts the
details, and runs a 12-question **Gate Check** that scores each application and
gives a verdict: **Good to Go**, **Can't Decide**, or **Don't Waste Time**. A
"knockout" flag marks applications that fail a critical check outright.

**How to use it (three tabs):**

- **Dashboard:** Number cards for total applications, awaiting review, reviewed,
  good-to-go, with-knockout, and average score.
- **New Applications:** Run the two-step scan:
    - **Step 1 — Read Inbox:** click **Read Inbox** to connect (read-only) and list
      new application emails.
    - **Step 2 — Parse & Run Gate Check:** click **Parse & Run** to extract the
      fields, scan the attached documents, and score the new applications.
- **Reviewed:** the applications already scored.
- In either list, click **View checks** on an application to expand its detail.
  Review and adjust the answers to each of the 12 questions; scores and verdict
  recalculate live as you edit. Click **Save edits & re-score** to keep changes,
  or force a verdict with the **Verdict Override** dropdown and **Save verdict**.

**Key settings:** Some answers are auto-filled from the application's documents;
others you can edit. Sensitive personal information is kept encrypted and not
displayed. One related setting — the maximum age for a pre-approval letter —
lives in Admin → Email / SMTP.

---

## Update Website

**What it does:** Helps you keep your public website's FAQ section current. Paste
in your existing FAQ, add new questions, and the tool drafts grounded answers,
then produces website-ready content you copy into WordPress. (News and Blog tabs
are placeholders, marked "coming up.")

**How to use it (four numbered steps):**

1. **Paste current FAQ HTML** and click **Load & parse** so the tool knows your
   existing questions and categories.
2. **Paste questions (Q1–Q15)** into the boxes. For each, mark it **Same**,
   **Different**, or **New** (and link it to an existing question if updating),
   and tick **gen** to include it.
3. **Generate & review** — click Generate to draft answers, then **Approve & Save**
   or **Discard**.
4. **Outputs** — copy the finished **FAQ HTML** and **Schema head** into WordPress.

**Key settings:** A right-side panel gives steps for pulling popular questions from
Ubersuggest. An editable **principles** list controls how answers are written (use
only real facts, stay accurate, keep it South-Carolina-specific). One principle is
locked and can't be removed.

---

## Ops Library

**What it does:** An internal reference hub for your workflows, standard operating
procedures, and templates — your process documents in one place.

**How to use it:**

- Pick a document from the left-hand list (currently **Application Workflow** and
  **Property Data**); it opens in the main pane.
- Each document has its own link for sharing or clean printing.

**Key settings:** More documents are planned. This is a read-only reference area.

---

## Unsubscribe

**What it does:** Shows the agents who opted out of your email outreach (your
suppression list) so they're never emailed again. It also surfaces email addresses
that bounced. This protects your sender reputation and keeps you compliant.

**How to use it:**

- Two cards at the top show how many agents are on **Do not email** and how many
  addresses have **Bounced**.
- The table lists each suppressed email, its status, when and how it was detected,
  and who (if anyone) re-added it.
- If someone was suppressed by mistake or asks to come back, click **Re-add →** on
  their row and enter a reason.

**Key settings:** Read-mostly; the only action is re-adding an agent, which is logged.

---

## Admin

The Admin area is organized into tabs. The ones a franchise owner uses day-to-day:

### Email / SMTP

Where you set up sending and receiving email, your PropStream login, and one
gate-check setting. The page splits **outgoing (marketing) email** from the
**incoming application inbox**, with a green/red banner showing whether each is
configured.

- Fill in the outgoing email fields (username, password, host, port, sender name,
  from-address, signature) and click **Update outgoing**. Click **Send test email**
  to confirm it works.
- Fill in the incoming inbox fields and click **Update incoming**; **Read inbox now**
  verifies the connection.
- Enter your **PropStream** username, password, and profile path, then **Save PropStream**.
- Set the **Gate Check** "pre-approval letter max age (days)" used by the
  Pre-Underwriting scanner.
- A **delay between emails** dropdown (30–45s, 45–60s recommended, 60–90s) paces
  your sending. Passwords are masked with a Reveal toggle.

### Master List

Manage your master agent list — the people you can email.

- Stat cards show **Total Agents**, **Offices**, **Missing Emails**, **Duplicates**,
  and when the list was last loaded.
- Click **Replace File** and choose a spreadsheet to replace the current master list
  (you'll confirm — this archives the old list and re-imports all agents).
- Set the **Match Strategy** (fuzzy name match recommended, or exact-name / email-only)
  and **Minimum Confidence Score** (75%–95%), then **Save Settings**.
- Search the list by name, email, or office.

### Templates

Edit your drip-email templates. They're organized into **Single AUC** (agents with
one listing) and **Multi AUC** (several listings), each with a 10-step contact
sequence (1st Contact through 10th Contact). Every change is version-tracked.

- Pick a template from the tree on the left.
- Edit the **Subject Line**, the **Opening Line** (unique to that template), and the
  **Shared Body** (shared across all templates of that type). Click **Save Template**
  — it saves as a new version.
- Insert personalization tags like `[FirstName]`, `[PropertyAddress]`, `[DaysToClose]`,
  `[ListPrice]`. Multi-AUC templates can also use `[PropertyTable]`, which lists all
  of an agent's active listings.
- Use **Version History** to view an earlier version and **Restore This Version**.

!!! note
    Editing the Shared Body changes all 10 templates of that type at once; only the
    Opening Line and Subject are unique per template.

### Reference Data

Maintains the behind-the-scenes lookup lists the app uses to interpret data —
**MLS statuses**, **Pre-qual terms**, and **Proof of funds terms**. These feed the
import and the pre-underwriting checks.

- Search and filter each list (active only / all / inactive only).
- Use the **Add New** box to add an entry (for MLS statuses you also pick a bucket
  like Under Contract or Dead Deal).
- You can **Deactivate** or **Reactivate** entries, but you cannot delete them.

### Send Utility (Mimic Real Send)

Re-send any past email to an address of your choice to confirm it renders correctly.
It does **not** record the send in Sent History, so you can test freely.

- Pick an agent from the dropdown (you'll see the last template and subject used).
- Enter the address you want the preview sent to, and click **Send Preview**.

### Backup & Restore

Back up your entire database and restore from a previous backup.

- Set the **Backup Directory** and **Save**.
- Click **Run Backup Now** to create a full backup; you'll see the row count,
  filename, and size.
- In **Backup History**, **Download** a backup, or **Restore** from one. Restoring
  **overwrites all current data** — you'll be warned and asked to type "RESTORE" to
  confirm. It cannot be undone.
- The **Encryption** panel shows whether the personal-data encryption key is
  configured; the key itself is never shown.

!!! info "Admin-only tabs"
    **National Reporting**, **Onboard Franchise**, **Logins**, **Docs Engine**, and
    **Test Suite** are reserved for the national administrator and the system owner.
    They're covered in the Technical and QA sections.
