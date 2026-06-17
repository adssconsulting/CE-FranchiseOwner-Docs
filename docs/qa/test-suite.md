# QA & Testing

!!! warning "Internal — for the system administrator"
    This page is for the CE FranchiseOwner administrator (the franchise's
    technical owner). If you're a franchise owner just here to learn the app,
    head back to the [User Guide](../marketing/feature-guides.md).

## What it is

The backend test suite (`backend/tests/test_suite.py`, 34 `test_*` functions) runs
end-to-end against a **live FastAPI backend** using `httpx.Client` — no mocks of the
app itself. It's designed to run repeatedly against the dev database **without side
effects**:

- Email sends use `simulate=True` (no real SMTP, no row writes).
- Every write endpoint round-trips (set → verify → restore).
- A session fixture captures a baseline row count and calls
  `/email-queue/_cleanup_test` afterward.
- Assertions are relational/range-based (e.g. "≥30 tables", ">100k rows") rather
  than brittle magic numbers, so they survive data evolution.

Each test's docstring is the plain-English "what was tested" line surfaced in the
Admin → Test Suite view.

## Coverage

- Health / environment and schema scale (table count, row volume)
- Dashboard and engagement summaries
- Agent matching and the unmatched-agent workflow
- Email-queue buckets, templates, stats, and simulated sends
- Properties / import stats
- Call tracker
- Admin settings round-trips
- Unsubscribe / bounce stats
- DB-query and row-count endpoints

## How to run

```bash
# from the repo root
python -m pytest backend/tests/test_suite.py -v

# inside the backend container
python -m pytest /app/tests/test_suite.py -v
```

Environment:

- `API_BASE` — backend URL (default `http://localhost:8001`)
- `TEST_AUTH_TOKEN` — a valid bearer token, required because `/api/admin/*` is auth-gated

## From the Admin UI

The **Admin → Test Suite** tab runs the same suite in-app:

- `POST /api/admin/test-suite/run` — runs it (passing a valid SC token as `TEST_AUTH_TOKEN`)
- `GET /api/admin/test-suite/list` — lists the tests and their docstrings
- `GET /api/admin/test-suite/report` — an HTML pass/fail report
