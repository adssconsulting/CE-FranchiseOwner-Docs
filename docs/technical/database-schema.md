# Database Schema

!!! warning "Internal — for the system administrator"
    This page is for the CE FranchiseOwner administrator (the franchise's
    technical owner). If you're a franchise owner just here to learn the app,
    head back to the [User Guide](../marketing/feature-guides.md).

PostgreSQL 16. The app connects as the non-superuser role `ce_app` so Row-Level
Security applies; `ce_web` is the superuser used for migrations. **36 tables**,
grouped by domain prefix below.

## Naming convention

Web tables follow `{domain}_{name}_{type}`, where the suffix signals row semantics:

- **`_fact`** — event / transaction rows (one row per occurrence: an email sent, a
  match made, a gate-check run).
- **`_dim`** — reference / lookup / dimension data (applications, templates, status codes).
- **`_log`** — append-only history (audit / activity trails).
- Tables without a suffix (`calltracker_log`, `system_log`, `app_login`,
  `alembic_version`) sit outside the convention.

The `territory` column (varchar) is the RLS branch key. On most tables it defaults
to `COALESCE(NULLIF(current_setting('app.territory'),''),'SC')`, so the session's
territory is stamped on insert and falls back to `SC` (the one real branch).

## Row-Level Security

Per request, the app sets `app.territory` (branch code) and `app.is_admin` (`'1'`
for the national admin). Two policy shapes exist:

- **`terr_isolation`** — `is_admin='1' OR territory = app.territory`. Data tables.
  The national admin sees all branches.
- **`terr_config`** — `territory = COALESCE(app.territory,'SC')`, **no admin bypass**,
  defaults to SC. The two per-branch config tables, so each branch edits only its own.

| Table | Policy |
|---|---|
| `agentoutreach_property_load_fact` | terr_isolation |
| `agentoutreach_agent_match_fact` | terr_isolation |
| `agentoutreach_email_send_log` | terr_isolation |
| `agentoutreach_master_agent_list` | terr_isolation |
| `agentoutreach_load_batch_dim` | terr_isolation |
| `agentoutreach_unsubscribe_list` | terr_isolation |
| `agentoutreach_email_bounce` | terr_isolation |
| `calltracker_match_fact` | terr_isolation |
| `engagement_fact` | terr_isolation |
| `admin_app_settings` | terr_config |
| `agentoutreach_email_template_dim` | terr_config |

All other tables have no RLS. `app_login` is deliberately global — authentication
runs before a territory is known.

## Desktop → Web rename mapping

Legacy desktop tables were renamed to the new convention (`backend/demo/rename_tables.sql`):

| Old (desktop) | New (web) |
|---|---|
| `agent_match_with_master` | `agentoutreach_agent_match_fact` |
| `master_list` | `agentoutreach_master_agent_list` |
| `master_matches` | `agentoutreach_master_match_fact` |
| `property_loads` | `agentoutreach_property_load_fact` |
| `load_inventory` | `agentoutreach_load_batch_dim` |
| `email_templates` | `agentoutreach_email_template_dim` |
| `email_template_history` | `agentoutreach_email_template_history` |
| `email_log` | `agentoutreach_email_send_log` |
| `unsubscribe_list` | `agentoutreach_unsubscribe_list` |
| `suppressed_log` | `agentoutreach_suppressed_log` |
| `saved_searches` | `agentoutreach_saved_search` |
| `cleansed_leads` | `agentoutreach_cleansed_lead` |
| `raw_auc_listings` | `agentoutreach_raw_auc_listing` |
| `raw_scrapes` | `agentoutreach_raw_scrape` |
| `cold_call_log` | `calltracker_log` |
| `cold_call_matches` | `calltracker_match_fact` |
| `ce_application_dim` | `preunderwriting_application_dim` |
| `ce_application_status` | `preunderwriting_application_status_dim` |
| `ce_application_receivables` | `preunderwriting_receivable_fact` |
| `ce_application_gate_check` | `preunderwriting_gate_check_fact` |
| `ce_application_gate_check_comments` | `preunderwriting_gate_check_comment` |
| `ce_application_api_calls` | `preunderwriting_api_call_log` |
| `ce_application_archive_json` | `preunderwriting_application_archive` |
| `ce_application_extraction_log` | `preunderwriting_extraction_log` |
| `mls_status_dim` | `admin_referencedata_mls_dim` |
| `prequal_term_dim` | `admin_referencedata_prequal_term_dim` |
| `proof_of_funds_term_dim` | `admin_referencedata_proof_of_funds_term_dim` |
| `app_settings` | `admin_app_settings` |
| `system_logs` | `system_log` |
| `db_migration_log` | `system_db_migration_log` |
| `demo_engagement_log` | `system_demo_engagement_log` |

(Sequence names still carry the old desktop names, e.g. `master_list_id_seq` —
only the tables were renamed.)

Two recurring patterns across the newer domains: **effective-dating**
(`eff_start_dttm`/`eff_end_dttm` + `curr_rec_ind`) to retain history without
deleting rows, and the **`nkey` / `nkey_hash`** natural-key pair to link related
rows across the preunderwriting and engagement tables.

---

## `agentoutreach_` — Agent outreach / email marketing (15 tables)

Scrape/import property listings, match listing agents to a master list, and email them.

### agentoutreach_raw_scrape
Raw scraped property/agent rows before cleansing.

| Column | Type | Null | Notes |
|---|---|---|---|
| id | integer | NO | PK |
| county, address, city, state, zip | varchar | YES | |
| list_price | numeric | YES | |
| mls_status | varchar | YES | |
| status_date | date | YES | |
| days_on_market | integer | YES | |
| agent_name, agent_email | varchar | YES | |
| mls_number | varchar | YES | |
| scraped_at | timestamp | YES | default now |

### agentoutreach_raw_auc_listing
Raw auction-listing scrape rows.

| Column | Type | Null | Notes |
|---|---|---|---|
| id | integer | NO | PK |
| county, property_address, status, action, agent, brokerage | varchar | YES | |
| status_date, listing_date, date_range_start, date_range_end | date | YES | |
| price, listing_price, avg_sale_price | numeric | YES | |
| mls_listing_num | varchar | YES | |
| scraped_at | timestamp | YES | default now |

### agentoutreach_cleansed_lead
Cleansed leads derived from raw scrapes.

| Column | Type | Null | Notes |
|---|---|---|---|
| id | integer | NO | PK |
| raw_scrape_id | integer | YES | FK → raw_scrape |
| agent_first_name, agent_last_name, agent_email, address, mls_number | varchar | YES | |
| list_price | numeric | YES | |
| days_to_close | integer | YES | |
| created_at | timestamp | YES | default now |

### agentoutreach_property_load_fact
Loaded property/listing records per import batch — the main working dataset. **RLS: terr_isolation.**

| Column | Type | Null | Notes |
|---|---|---|---|
| id | integer | NO | PK |
| load_batch_id | varchar | NO | batch grouping key |
| county_id, county_name | varchar | NO | |
| address, city, state, zip, property_type | varchar | YES | |
| est_value, mls_amount, lien_amount | numeric | YES | |
| mls_status | varchar | YES | |
| mls_date | date | YES | |
| mls_agent_name, mls_agent_email | varchar | YES | |
| mls_brokerage_name, mls_brokerage_phone | varchar | YES | |
| date_added_to_list | timestamp | YES | |
| source_file, do_not_mail, property_status, method_of_add | varchar | YES | |
| curr_rec_ind | char | YES | default 'Y' (soft-delete) |
| insrt_dttm | timestamp | YES | default now |
| territory | varchar | YES | RLS key, default SC |

### agentoutreach_load_batch_dim
Inventory of import batches (one row per file/county load). **RLS: terr_isolation.**

| Column | Type | Null | Notes |
|---|---|---|---|
| id | integer | NO | PK |
| load_batch_id, county_id, county_name | varchar | NO | |
| source_filename, notes | varchar | YES | |
| record_count | integer | YES | default 0 |
| load_status | varchar | YES | default 'pending' |
| loaded_at | timestamp | YES | default now |
| territory | varchar | YES | RLS key, default SC |

### agentoutreach_master_agent_list
Master list of agents (golden/reference records). **RLS: terr_isolation.**

| Column | Type | Null | Notes |
|---|---|---|---|
| id | integer | NO | PK |
| email, first_name, last_name, phone, office_name, license_number | varchar | YES | |
| curr_rec_ind | char | YES | default 'Y' |
| insrt_dttm | timestamp | YES | default now |
| load_batch_id | varchar | YES | |
| territory | varchar | YES | RLS key, default SC |

### agentoutreach_agent_match_fact
Results of matching MLS agents against the master list (golden-record workflow). **RLS: terr_isolation.**

| Column | Type | Null | Notes |
|---|---|---|---|
| id | integer | NO | PK |
| mls_agent_name, mls_brokerage | varchar | YES | |
| matched_email, matched_first, matched_last, matched_office | varchar | YES | |
| confidence_score | numeric | YES | |
| match_type | varchar | YES | default 'AUTO' |
| match_status | varchar | YES | default 'PENDING' |
| is_verified, is_golden, is_rejected, curr_rec_ind | char | YES | default 'N' / 'Y' |
| verified_by | varchar | YES | |
| verified_dttm, created_at | timestamp | YES | |
| territory | varchar | YES | RLS key, default SC |

### agentoutreach_master_match_fact
Cleansed-lead → master-agent matches.

| Column | Type | Null | Notes |
|---|---|---|---|
| id | integer | NO | PK |
| cleansed_lead_id | integer | YES | FK → cleansed_lead |
| agent_email, agent_first_name, agent_last_name, office_name | varchar | YES | |
| confidence_score | numeric | YES | |
| matched_at | timestamp | YES | default now |

### agentoutreach_email_template_dim
Per-territory email templates (current version). **RLS: terr_config (no admin bypass).** Unique on `(template_number, territory)`.

| Column | Type | Null | Notes |
|---|---|---|---|
| id | integer | NO | PK |
| template_number | integer | NO | |
| template_name, subject_line, last_updated_by | varchar | YES | |
| opening_line, shared_body | text | YES | |
| current_version | integer | YES | default 1 |
| updated_at | timestamp | YES | default now |
| territory | varchar | YES | RLS key, default SC |

### agentoutreach_email_template_history
Append-only version history of templates.

| Column | Type | Null | Notes |
|---|---|---|---|
| id | integer | NO | PK |
| template_number, version_number | integer | NO | |
| template_name, subject_line, saved_by, notes | varchar | YES | |
| opening_line, shared_body | text | YES | |
| saved_at | timestamp | YES | default now |

### agentoutreach_email_send_log
Log of every email sent to agents. **RLS: terr_isolation.**

| Column | Type | Null | Notes |
|---|---|---|---|
| id | integer | NO | PK |
| agent_email, sender_email, agent_name, property_address, subject, status, template_name, batch_name, auc_bucket | varchar | YES | |
| body | text | YES | |
| template_version, contact_count | integer | YES | |
| mls_amount | numeric | YES | |
| is_test | smallint | YES | default 0 |
| sent_at | timestamp | YES | default now |
| territory | varchar | YES | RLS key, default SC |

### agentoutreach_email_bounce
Hard-bounce detections from the inbox poller. **RLS: terr_isolation.**

| Column | Type | Null | Notes |
|---|---|---|---|
| id | integer | NO | PK |
| agent_email | varchar | NO | |
| agent_name, bounce_type, email_subject, status, cleared_by | varchar | YES | default type 'hard', status 'bounced' |
| diagnostic, cleared_reason | text | YES | |
| bounced_at, cleared_at, created_at | timestamp | YES | |
| territory | varchar | YES | RLS key, default SC |

### agentoutreach_unsubscribe_list
Suppressed / unsubscribed agents. **RLS: terr_isolation.** Unique on `(agent_email, territory)`.

| Column | Type | Null | Notes |
|---|---|---|---|
| id | integer | NO | PK |
| agent_email | varchar | NO | |
| agent_name, detection_method, keyword_matched, email_subject, status, readded_by | varchar | YES | |
| readded_reason | text | YES | |
| unsubscribed_at, readded_at, created_at | timestamp | YES | |
| territory | varchar | YES | RLS key, default SC |

### agentoutreach_suppressed_log
Append-only log of agents suppressed from a send (e.g. no new addresses).

| Column | Type | Null | Notes |
|---|---|---|---|
| id | integer | NO | PK |
| agent_email, agent_name, reason | varchar | YES | reason default 'no_new_addresses' |
| suppressed_at | timestamp | YES | default now |

### agentoutreach_saved_search
Named saved searches in the outreach UI.

| Column | Type | Null | Notes |
|---|---|---|---|
| id | integer | NO | PK |
| search_name | varchar | NO | |
| sort_order | integer | NO | default 0 |
| created_at | timestamp | YES | default now |

---

## `preunderwriting_` — Commission-advance application processing (8 tables)

Intake and pre-underwriting of commission-advance applications, including the
automated 12-question gate-check engine and LLM-based document extraction.

### preunderwriting_application_dim
Core application record (agent, company, property, deal, document URLs). Contains
**sensitive PII** — SSN, bank routing/account, driver's-license — stored encrypted.
Key columns include: `nkey` / `nkey_hash` (natural key + hash), agent and company
identity fields, bank details, property/deal fields (`final_price`,
`total_net_commission`, `cash_advance_requested`, `ratification_date`,
`projected_settlement_date`), document URLs (`pdf_url_purchase_contract`,
`pdf_url_compensation_agreement`, `pdf_url_prequal_proof_of_funds`,
`pdf_url_mls`, `pdf_url_drivers_license`, …), and the originating email
(`email_received_dttm`, `email_from`, `raw_email_body`). ~70 columns total.

### preunderwriting_application_status_dim
Application status history (effective-dated; current row flagged `curr_rec_ind=1`).

| Column | Type | Null | Notes |
|---|---|---|---|
| id | integer | NO | PK |
| nkey, nkey_hash, status, changed_by | varchar | NO/YES | |
| application_id | integer | NO | FK → application_dim |
| status_notes | text | YES | |
| eff_start_dttm, eff_end_dttm, inserted_dttm | timestamp | YES | |
| curr_rec_ind | integer | YES | default 1 |

### preunderwriting_gate_check_fact
Automated gate-check scoring run. One row holds per-question scores **Q1–Q12**
(with sources, modifiers, and net scores), knockout flags (`has_knockout_ind`,
`knockout_questions`), `total_score`, `verdict`, and a manual override block
(`override_flag`, `override_verdict`, `override_comment`, `override_by`,
`override_dttm`). ~60 columns. FK `application_id` → application_dim.

### preunderwriting_gate_check_comment
Manual override comments per gate-check question.

| Column | Type | Null | Notes |
|---|---|---|---|
| id | integer | NO | PK |
| nkey_hash | varchar | NO | |
| question_number | smallint | NO | |
| override_comment | text | NO | |
| created_by | varchar | YES | default 'sc.manager' |
| created_dttm, updated_dttm | timestamp | YES | default now |

### preunderwriting_receivable_fact
Commission receivables / advance requests per application (effective-dated).

| Column | Type | Null | Notes |
|---|---|---|---|
| id | integer | NO | PK |
| nkey, nkey_hash | varchar | NO | |
| application_id, receivable_seq | integer | NO | FK → application_dim |
| advance_requested, total_net_commission | numeric | YES | |
| sell_entire_or_partial | varchar | YES | |
| email_triggered_ind, curr_rec_ind | integer | YES | default 1 |
| eff_start_dttm, eff_end_dttm, inserted_dttm | timestamp | YES | |

### preunderwriting_api_call_log
Log of LLM/API extraction calls per application question (prompt, output, tokens, cost, overrides).

| Column | Type | Null | Notes |
|---|---|---|---|
| id | integer | NO | PK |
| nkey, nkey_hash, question_ref, api_model_called, extracted_value, confidence_level, override_value, override_by, call_status | varchar | | |
| application_id, tokens_input, tokens_output | integer | | FK → application_dim |
| pdf_direct_url, prompt_used, llm_output_raw, error_message | text | YES | |
| cost_usd | numeric | YES | |
| human_override_ind, curr_rec_ind | integer | YES | |
| override_dttm, eff_start_dttm, eff_end_dttm, inserted_dttm | timestamp | YES | |

### preunderwriting_extraction_log
Append-only log of field-extraction attempts (source, parsed value, confidence).

| Column | Type | Null | Notes |
|---|---|---|---|
| id | integer | NO | PK |
| application_nkey_hash, extraction_source | varchar | NO | |
| q_number | smallint | NO | |
| raw_extracted, parsed_value, bucket, dim_table, confidence, overwritten_by_user, error_message | varchar | YES | |
| snippet | text | YES | |
| confidence_score | numeric | YES | |
| parse_duration_ms, dim_row_id | integer | YES | |
| curr_rec_ind | char | YES | default 'Y' |
| eff_start_dt, eff_end_dt | timestamp | | |

### preunderwriting_application_archive
Archived application snapshots (JSON payload) when an application is archived.

| Column | Type | Null | Notes |
|---|---|---|---|
| id | integer | NO | PK |
| nkey_hash, archive_reason, archived_by | varchar | NO | |
| payload | text | NO | JSON snapshot |
| archived_at | timestamp | YES | default now |

---

## `calltracker_` — Cold-call tracking (2 tables)

### calltracker_log
Cold-call activity log (one row per agent being called).

| Column | Type | Null | Notes |
|---|---|---|---|
| id | integer | NO | PK |
| agent_name | varchar | NO | |
| email, phone, call_status, outcome | varchar | YES | call_status default 'Not called' |
| notes | text | YES | |
| past_deals, match_confidence | integer | YES | |
| followup_date | date | YES | |
| called_at, created_at | timestamp | YES | |

### calltracker_match_fact
Matches of imported call-list agents to master records. **RLS: terr_isolation.**

| Column | Type | Null | Notes |
|---|---|---|---|
| id | integer | NO | PK |
| import_agent_name | varchar | NO | |
| import_email, import_phone, master_first, master_last, master_email, master_phone, master_office, match_status, reviewed_by, readded_by, load_batch_id | varchar | YES | match_status default 'PENDING' |
| readded_reason | text | YES | |
| past_deals, confidence_score | integer | YES | |
| curr_rec_ind | char | YES | default 'Y' |
| reviewed_at, readded_at, created_at | timestamp | YES | |
| territory | varchar | YES | RLS key, default SC |

---

## `admin_` — Settings & reference data (4 tables)

### admin_app_settings
Per-territory application settings (key/value). **RLS: terr_config.** Unique on `(setting_key, territory)`.

| Column | Type | Null | Notes |
|---|---|---|---|
| id | integer | NO | PK |
| setting_key | varchar | NO | e.g. `smtp.host`, `mls_api.base_url` |
| setting_value | text | YES | sensitive values encrypted/base64 |
| updated_at | timestamp | YES | default now |
| territory | varchar | YES | RLS key, default SC |

### admin_referencedata_mls_dim
MLS status normalization lookup (raw → normalized, gate-check bucket).

| Column | Type | Null | Notes |
|---|---|---|---|
| id | integer | NO | PK |
| status_raw, status_normalized, display_label | varchar | NO | |
| gate_check_bucket | text | NO | |
| mls_sources, notes | varchar | YES | |
| curr_rec_ind | char | YES | default 'Y' |
| created_at | timestamp | YES | default now |

### admin_referencedata_prequal_term_dim
Pre-qualification term normalization lookup.

| Column | Type | Null | Notes |
|---|---|---|---|
| id | integer | NO | PK |
| term_normalized, term_display | varchar | NO | |
| applies_to | text | NO | default 'both' |
| curr_rec_ind | char | YES | default 'Y' |
| created_at | timestamp | YES | default now |

### admin_referencedata_proof_of_funds_term_dim
Proof-of-funds term normalization lookup.

| Column | Type | Null | Notes |
|---|---|---|---|
| id | integer | NO | PK |
| term_normalized, term_display | varchar | NO | |
| curr_rec_ind | char | YES | default 'Y' |
| created_at | timestamp | YES | default now |

---

## `engagement_` — Agent engagement tracking (2 tables)

### engagement_fact
One row per engagement event (call, email reply, etc.). **RLS: terr_isolation.**

| Column | Type | Null | Notes |
|---|---|---|---|
| engagement_id | bigint | NO | PK |
| engagement_nkey, engagement_nkey_hash, agent_name | varchar | NO | |
| agent_email, channel, source, source_ref | varchar | YES | |
| agent_master_id, engagement_type_id | integer | | FK → master_agent_list / engagement_type_dim |
| details | text | YES | |
| engagement_date, inserted_dttm | timestamp | YES | |
| curr_rec_ind | char | YES | default 'Y' |
| territory | varchar | YES | RLS key, default SC |

### engagement_type_dim
Lookup of engagement types/values.

| Column | Type | Null | Notes |
|---|---|---|---|
| engagement_type_id | integer | NO | PK |
| utility, engagement_value, code | varchar | NO | |
| sort_order | integer | YES | default 0 |
| curr_rec_ind | char | YES | default 'Y' |

---

## `system_` — System / operational (3 tables)

### system_log
Application log records.

| Column | Type | Null | Notes |
|---|---|---|---|
| id | integer | NO | PK |
| level, module | varchar | YES | |
| message | text | YES | |
| created_at | timestamp | YES | default now |

### system_db_migration_log
Audit trail of applied DB migrations (snapshot of SQL run).

| Column | Type | Null | Notes |
|---|---|---|---|
| id | integer | NO | PK |
| statements_count | integer | YES | |
| tables_affected, applied_by | varchar | YES | |
| sql_snapshot | text | YES | |
| applied_at | timestamp | YES | default now |

### system_demo_engagement_log
National-report / demo engagement data (created empty in prod so the admin report returns zeros instead of erroring).

| Column | Type | Null | Notes |
|---|---|---|---|
| id | integer | NO | PK |
| territory, ts, agent, property_trigger, classification, classification_note | varchar | YES | territory default 'SC' |
| response | text | YES | |
| sort_order | integer | YES | |

---

## Standalone tables

### app_login
Data-driven franchise logins (one row per username). **Global, NO RLS** —
authentication happens before a territory is known. Passwords stored only as
salted PBKDF2 hashes.

| Column | Type | Null | Notes |
|---|---|---|---|
| id | integer | NO | PK |
| login_id | varchar | NO | username, unique |
| franchise_name | varchar | YES | display name |
| territory | varchar | YES | branch code; NULL = national admin |
| is_admin | boolean | YES | default false |
| password_hash | text | YES | `pbkdf2_sha256:…`; NULL = legacy auth path |
| curr_rec_ind | char | YES | default 'Y' |
| created_at | timestamp | YES | default now |

### alembic_version
Alembic migration version tracker.

| Column | Type | Null | Notes |
|---|---|---|---|
| version_num | varchar | NO | PK; current migration revision |
