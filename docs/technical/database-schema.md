# Database Schema

!!! info "Placeholder"
    Run the **Docs Engine** tab to regenerate every table reference
    below from the current `init_database()` in `core/database.py`
    and the migration helpers.

## Conventions

- `curr_rec_ind` (`'Y'` / `'N'`) — soft-delete flag. Rows are never
  `DELETE`'d; the historical row is set to `'N'` and a new `'Y'` row
  is inserted.
- `load_batch_id` — groups rows by import session. Format
  `INIT_YYYYMMDD_HHMMSS` for property loads, `YYYYMMDD_HHMMSS` for
  master-list loads.
- `county_id` — `SC_<COUNTYNAME>_AUC` (e.g. `SC_CHARLESTON_AUC`).
- Sensitive `app_settings` values (any key containing `password`,
  `secret`, `token`, or `api_key`) are base64-encoded at rest.

## Tables

_Every table from `init_database()` will be enumerated here by the
Docs Engine run._
