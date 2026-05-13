# Database Schema

The schema is defined imperatively by `core.database.init_database()`
(idempotent `CREATE TABLE IF NOT EXISTS`) and evolved by the
`_migrate_*` helpers in the same file. The dev-only **DB Migrations**
admin tab compares dev's live schema against prod and applies the
delta on demand.

## Conventions

| Convention | Meaning |
|---|---|
| `curr_rec_ind` (CHAR(1), `'Y'`/`'N'`) | Soft-delete flag. Rows are never `DELETE`'d; the historical row is set to `'N'` and a new `'Y'` row is inserted. Active rows are `'Y'`. |
| `load_batch_id` (VARCHAR(50)) | Groups rows by import session. `INIT_YYYYMMDD_HHMMSS` for property loads, `YYYYMMDD_HHMMSS` for master-list loads. |
| `county_id` (VARCHAR(50)) | `SC_<COUNTYNAME>_AUC`, e.g. `SC_CHARLESTON_AUC`. Derived from PropStream export filenames. |
| `is_golden` (CHAR(1)) | Human-verified match. `Y` = the canonical "this PropStream listing-name → this master agent" record. |
| `match_status` (VARCHAR(20)) | `PENDING` / `APPROVED` / `REJECTED`. New source of truth for review state (replaced the older boolean trio). |
| PII columns | `VARBINARY(500)` — Fernet-encrypted at rest using the key stored in `app_settings.pii_encryption_key`. Decrypted at read time by `core.crypto.decrypt_pii`. |
| Sensitive `app_settings` values | Any key whose name contains `password`, `secret`, `token`, or `api_key` is base-64-encoded at rest by `core.settings_manager`. |

## Lead Engine tables

### `raw_scrapes`
Raw rows captured from PropStream scrape runs.
| Column | Type | Notes |
|---|---|---|
| `id` | INT PK | |
| `county` | VARCHAR(100) | |
| `address`, `city`, `state`, `zip` | VARCHAR | |
| `list_price` | DECIMAL(12,2) | |
| `mls_status` | VARCHAR(50) | |
| `status_date` | DATE | |
| `days_on_market` | INT | |
| `agent_name`, `agent_email`, `mls_number` | VARCHAR | |
| `scraped_at` | TIMESTAMP default `NOW()` | |

### `cleansed_leads`
Normalised lead rows derived from `raw_scrapes`. FK to `raw_scrape_id`.

### `master_matches` *(legacy)*
First-generation match table. Kept for backward compat; new code
writes to `agent_match_with_master`. A one-time migration in
`_migrate_master_matches_to_amwm` populates the new table from this
one if the new table is empty.

### `agent_match_with_master`
**Primary** agent matching table. Every row pairs a PropStream
listing-agent name with a master-list agent.
| Column | Type | Notes |
|---|---|---|
| `id` | INT PK | |
| `mls_agent_name` | VARCHAR(150) | |
| `mls_brokerage` | VARCHAR(200) | |
| `matched_email` | VARCHAR(150) | |
| `matched_first`, `matched_last`, `matched_office` | VARCHAR | |
| `confidence_score` | DECIMAL(5,2) | rapidfuzz score |
| `match_type` | VARCHAR(20) default `AUTO` | `AUTO` / `MANUAL_SEARCH` / `MANUAL_EMAIL` / `SKIP` |
| `match_status` | VARCHAR(20) default `PENDING` | `PENDING` / `APPROVED` / `REJECTED` |
| `is_verified`, `is_golden`, `is_rejected`, `curr_rec_ind` | CHAR(1) | |
| `verified_by`, `verified_dttm` | | who flipped the row, when |
| `created_at` | TIMESTAMP | |

### `property_loads`
PropStream property data imported via the Initial Load wizard (18
PropStream columns, plus `county_id`, `source_file`,
`load_batch_id`, `curr_rec_ind`, `insrt_dttm`).

### `load_inventory`
One row per import file per batch. Tracks `county_id`, `county_name`,
`source_filename`, `record_count`, `load_status` (`pending` /
`loaded` / `failed`), `loaded_at`, free-form `notes`.

### `master_list`
SC real-estate agent master list with `phone` column for cold-call
workflow.

### `saved_searches`
PropStream saved-search names (one per county) with `sort_order` for
the loop order in the Daily Job.

### `email_templates` and `email_template_history`
- `email_templates` — current template content with `current_version`.
- `email_template_history` — append-only audit log; one row per save
  with the full content snapshot.

### `email_log`
Sent email history. Every successful send (`ok=True`) writes one row
here with `sent_at=NOW()`. Test sends (`is_test=1`) are excluded from
the Sent History tab's main list but shown in *Test Runs*.

### `system_logs`
General audit log. Columns: `level`, `module`, `message`,
`created_at`. `module` values include `startup`, `db_warn`,
`backup_manager`, `batch_progress`, `migration`, `ref_data_change`,
`doc_engine`, plus screen-specific tags.

### `app_settings`
Key/value config: `(setting_key, setting_value, updated_at)`. Used
by every screen's *Save* button.

### `cold_call_matches`
Staged cold-call matches awaiting Review Call Matches approval.
Tracks `import_agent_name`, `match_status`, `reviewed_by`,
`reviewed_at`, plus re-add audit columns (`readded_by`,
`readded_at`, `readded_reason`).

### `cold_call_log`
Active call list. Unique key on `agent_name`. Columns include
`call_status`, `outcome`, `notes`, `followup_date`, `called_at`,
`match_confidence`.

### `unsubscribe_list`
Suppression list. Unique key on `agent_email`. Populated by the
IMAP poller. Columns: `agent_email`, `agent_name`,
`unsubscribed_at`, `detection_method`, `keyword_matched`,
`email_subject`, `status` (`suppressed` / `re-added`),
`readded_at`, `readded_by`, `readded_reason`.

## Pre-Underwriting tables

### `ce_application_dim`
The parsed application. Most columns are `VARBINARY(500)` — Fernet
ciphertext over UTF-8 plaintext. Indexed by `nkey_hash` for fast
lookup; `email_received_dttm` indexed for chronological browsing.

Highlights:

- Agent: first/last name, SSN, email, cell phone, home address,
  city/state/zip — all encrypted.
- Company: name, broker rep name (encrypted), address, phone
  (encrypted), broker email (encrypted).
- Financial: bank name, routing number, account number, DL number
  — all encrypted.
- Property: address, city/state/zip, seller/purchaser names,
  reference number.
- Lender: loan officer (encrypted), lender phone (encrypted), final
  price, ratification date, projected settlement date.
- Pipeline: `num_pending_settlements`, `num_settled_last_12_months`,
  `num_active_listings`, `prior_year_income`.
- PDFs: 7 nullable URL columns (`pdf_url_purchase_contract`, …),
  one per known document type.
- Audit: `inserted_dttm`, `updated_dttm`,
  `email_received_dttm`, `raw_email_body` (long text, plaintext).

### `ce_application_receivables`
Multiple receivables per application (sequence). FK by
`application_id`.

### `ce_application_api_calls`
Audit log of every Anthropic call: model, prompt, raw LLM output,
extracted value, confidence level, token counts, cost in USD,
override fields. Indexed by `application_id` and `question_ref`.

### `ce_application_gate_check`
The 12 question scores plus the aggregate verdict. Includes
modifier fields for Q7 / Q9 (down-payment ratio, deal-type modifier),
a `has_knockout_ind`, the `knockout_questions` (concatenated refs),
and the final `verdict` string.

### `ce_application_status`
Effectively-dated status history per application. `eff_start_dttm` /
`eff_end_dttm` / `curr_rec_ind` triplet — the active row (`Y`) is
the current status.

## Migrations

Schema evolution is handled in-code, not in `.sql` migration files.
`init_database()` ends by calling `_migrate_*` helpers that
`SHOW COLUMNS` and conditionally `ALTER TABLE ... ADD COLUMN` when
the live schema is missing a column. The DB Migrations admin tab
(dev-only) drives operator-confirmed migrations against prod and
records each apply in `ce_franchiseowner.db_migration_log`.
