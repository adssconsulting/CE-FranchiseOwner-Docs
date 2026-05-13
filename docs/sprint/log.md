# Release Notes / Sprint Log

Newest entry first. Each Docs Engine run appends a new dated section
to the top of this file.

---

## 2026-05-13 — Docs Engine first regeneration

First end-to-end run of the Docs Engine workflow. All 9 content pages
on this site were regenerated from the live source code in
`CE-FranchiseOwner@main`. Highlights of what landed in the source
repo since the scaffold-only first commit:

### Admin shell
- **Docs Engine** tab added to the Admin notebook (`gui/docs_tab.py`),
  with a Copy Prompt button that drives this very regeneration.
- **`?` help button** placed on every Admin tab; opens the
  corresponding page on this site via `utils.help_links.open_help()`.

### Lead Engine
- **Email Queue** redesign:
  - Real-time **Search agent** filter above the agent list (case-
    insensitive substring match, debounced via Tk var trace).
  - **AUC bucket overlap fixed.** Buckets are now non-overlapping:
    `>10`, `8-10`, `5-7`, `3-5` (≤4 only), `2-3` (=2 only), `1`.
    Every agent maps to exactly one bucket.
  - **Data-quality filters** added to every `property_loads` query —
    `address` and `city` must be non-null and non-empty.
  - **Address deduplication** via `COUNT(DISTINCT address, city,
    state)` in count queries; `GROUP BY` in the per-email property
    table so an agent never sees the same address twice in their
    drip email.
  - **Persistent batch progress banner** at the top of the Email
    Queue. Writes `RUNNING|name|sent|total|failed` to `system_logs`
    after each send; flips to `COMPLETE|...` at the end. The banner
    polls every 5 s, shows orange during a run and green for 10
    minutes after completion, even if the user navigates away and
    back. Dismissable with an X button. All polling callbacks
    `winfo_exists()`-guarded.

### Review Matches
- Full screen rewrite. Old card-per-agent / canvas-scroll layout
  removed entirely; the screen is now two sections:
  1. **Step 1 — Download Review File** — generates a 3-sheet Excel
     (100% / 90-99% / 80-89% Match) with the DECISION column
     pre-filled `Y` on the high-confidence sheets and empty on
     80-89%. Y/N data-validation dropdowns are attached.
  2. **Step 2 — Upload & Apply Decisions** — reads all three sheets,
     validates every row has Y or N, runs two batched UPDATE
     statements. The UPDATEs are gated on `curr_rec_ind='Y'`.

### Backup & Restore (Admin)
- Auto-backup runs on every production startup if the last backup
  is older than 24 h. Output zipped into
  `Documents\CommissionExpress\Backups\`; max 30 retained.
- Manual *Run Backup Now* button; per-row *Restore* and *Delete*;
  Restore auto-restarts the app via `os.execv`.

### Call Tracker
- New **Add / Re-add** 4th tab. Lists parked (rejected) cold-call
  matches; clicking *Re-add* prompts for a reason via
  `simpledialog.askstring`, then runs the same DB writes the Admin
  Call Tracker uses (flip `match_status='APPROVED'`, stamp
  `readded_by`/`readded_at`/`readded_reason`, insert into
  `cold_call_log` with `call_status='Not called'`).

### Stability
- **`winfo_exists()` guards** added to every async callback across
  `gui/screens/` — `self.after()` lambdas, `bind_all` global
  handlers, and worker-thread → main-thread method handoffs. The
  recurring `_tkinter.TclError: invalid command name` tracebacks
  from screens that destroy mid-callback are eliminated.
- **DB Migrations** records every applied migration to
  `ce_franchiseowner.db_migration_log` with statements count and
  affected tables.

### Documentation infrastructure
- `CE-FranchiseOwner-Docs` repository scaffolded with MkDocs
  Material, indigo palette, light + dark scheme toggle.
- `.github/workflows/deploy.yml` auto-publishes `main` to GitHub
  Pages via `mkdocs gh-deploy --force` on every push, with
  `concurrency: docs-deploy / cancel-in-progress: true` so rapid
  pushes don't pile up.

---

## 2026-05-13 — Docs site scaffolded

Initial MkDocs Material scaffold created. GitHub Actions workflow
`.github/workflows/deploy.yml` added to auto-publish `main` to
GitHub Pages on every push. Placeholder pages added for every nav
entry.

Source project: [`CE-FranchiseOwner`](https://github.com/adssconsulting/CE-FranchiseOwner)
— Docs Engine tab in the Admin window now ships the prompt that
regenerates these pages.
