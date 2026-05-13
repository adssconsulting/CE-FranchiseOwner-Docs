# API & Integrations

Every external system the app talks to, where the wire-up lives in
the code, and which `app_settings` keys configure it.

## PropStream (browser automation)

- **Module.** `scraper/propstream_scraper.py` (Selenium, current
  production path) and `scraper/propstream_scraper_playwright.py`
  (Playwright, experimental).
- **How.** `webdriver-manager` resolves the ChromeDriver for the
  installed Chrome version. Chrome is launched with
  `--user-data-dir=<persistent profile>` plus anti-detection flags
  so PropStream's session cookies survive between scrape runs. If
  the session has expired, the scraper types the username and
  password from `app_settings` with randomised 0.05-0.15s per-key
  delays.
- **Settings keys** (`app_settings`):
  - `propstream.username`
  - `propstream.password` *(base-64 encoded)*
  - `propstream.profile_path` *(absolute Windows path to the
    Chrome user-data directory)*
- **Failure modes.** "Chrome profile not found" if the path
  doesn't exist; "Session expired, attempting auto-login" if the
  scraper detects the login page after navigation; otherwise
  prints PropStream's DOM error verbatim.

## SMTP (outgoing email — Rackspace)

- **Module.** `email_engine/email_sender.py`. Single entry point
  used by every screen that sends email. Wraps `smtplib.SMTP_SSL`
  / `smtplib.SMTP` with STARTTLS depending on port.
- **Behaviour.** Loads `smtp.*` settings, appends the
  `smtp.signature` to the body with a `---` separator, sends, and
  writes a row to `email_log` on success (with `sent_at=NOW()`,
  `is_test=0` for production sends, `is_test=1` for test sends).
  Also writes a file-line to `data/logs/email_sends.log` for an
  offline audit trail.
- **Settings keys.**
  - `smtp.sender_name` — display name on outgoing email
  - `smtp.sender_email` — From: address
  - `smtp.host` — typically `secure.emailsrvr.com`
  - `smtp.port` — `587`
  - `smtp.username` — full address; same as IMAP username
  - `smtp.password` *(base-64 encoded)*
  - `smtp.delay` — pacing label used by the batch sender
  - `smtp.signature` — multi-line text appended to every send

## IMAP (unsubscribe poller + application inbox — Rackspace)

The same Rackspace mailbox credentials power **two** distinct
pollers:

### Unsubscribe poller

- **Module.** `email_engine/unsubscribe_poller.py`.
- **Schedule.** Daemon thread checks at 12:00 and 16:30 local
  (see `CHECK_TIMES` in `main.py`).
- **Behaviour.** Searches the inbox for messages containing any of
  `stop`, `unsubscribe`, `remove me`, `opt out`,
  `do not contact`, `do not email` in the subject or body, derives
  the agent email from the reply chain, and `INSERT IGNORE` into
  `unsubscribe_list`. Future drip sends skip any address in that
  table.
- **Settings.** Reuses `smtp.username` and `smtp.password`. Host
  is `secure.emailsrvr.com`, port 993, TLS.

### Application-email poller (Pre-Underwriting)

- **Module.** `gui/screens/pre_underwriting_screen.py` (Step 1
  *Read inbox* button).
- **Behaviour.** Pulls unread messages from a dedicated mailbox
  matching the CE application form's `From:` address, passes each
  HTML body to `core.email_parser.parse_application_email()`,
  encrypts every PII field with `core.crypto.encrypt_pii()` using
  the Fernet key in `app_settings.pii_encryption_key`, and inserts
  the result into `ce_application_dim` with the `nkey_hash` index.
- **Settings.** Reuses `smtp.username` / `smtp.password`. The
  dedicated mailbox is typically a sub-folder or a separate
  Rackspace mailbox; configuration lives in `app_settings` keys
  prefixed `imap.application_*`.

## Anthropic API (Claude Haiku 4.5)

- **Module.** `core/llm_scanner.py`.
- **Purpose.** Two targeted PDF reads used by the gate check:
  - **Q4** — `scan_q4_contingencies(pdf_url)` — read a purchase
    contract for any open/active contingencies that could block
    closing.
  - **Q8** — `scan_q8_prequal(pdf_url)` — read a pre-qual letter
    or proof-of-funds PDF for buyer-financing readiness.
- **Wire.** Standard Messages API call to
  `https://api.anthropic.com/v1/messages` with model
  `claude-haiku-4-5` and `anthropic-beta: pdfs-2024-09-25` to send
  PDFs as `document` content blocks. 120-second timeout.
- **Cost tracking.** Each call returns `tokens_in`, `tokens_out`,
  and `cost_usd` (priced at $1 / 1M input, $5 / 1M output for
  Haiku 4.5). Every call is logged to `ce_application_api_calls`
  with the raw response.
- **Settings.** `app_settings.anthropic_api_key` *(base-64
  encoded)*. If missing, both functions short-circuit and return
  `{status: 'NO_API_KEY', reason: 'API key not configured'}` —
  the rest of the gate check still runs.

## PDF download / classification

- **Module.** `core/pdf_extractor.py`.
- **`classify_pdf_url(url)`** parses the Gravity-Forms
  `?gf-download=...` URL parameter, extracts the filename
  fragment, and maps it via `PDF_TYPE_PATTERNS` to one of
  `mls` / `purchase` / `prequal` / `compensation` / `license` /
  `misc` / `unknown`. Pure local string work — no network.
- **`fetch_pdf_bytes(url)`** downloads the PDF body in memory
  (`requests.get`). License PDFs are explicitly gated upstream so
  this function never sees one.
- **`extract_text_from_bytes(b)`** uses `pdfplumber` to extract
  text per page and concatenate, used for the local-only
  heuristic field parsers in `core/field_parsers/`.

## License key activation *(disabled)*

`gui/license_window.py` ships a one-time license-key activation
screen, currently bypassed for demo. To re-enable: wrap
`_run_login()` in `main.py` with `LicenseWindow(on_activate=…)`.
