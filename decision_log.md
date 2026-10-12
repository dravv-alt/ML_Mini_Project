# Decision log

## Phase 0 — 2026-10-12

- The supplied dataset remains in its ZIP and is read directly; no external data is used.
- Fixed random seed: `20261012`.
- Large Parquet tables, serialized models, and the supplied ZIP are intentionally Git-ignored. Small audit tables, reports, figures, configuration, and the notebook are versioned.
- The GitHub remote was empty. The workspace was initialized on `main` without overwriting prior history.
- The initial notebook draft is being executed sequentially and will be refactored into small inspectable cells before the final checkpoint.

## Phase 1 — 2026-10-12

- All seven required CSV schemas matched the columns specified by `context.md`.
- Raw tables were inspected independently before relational merges.
- `studentVle` contains duplicate composite event keys; these are valid multiple recorded rows at the same student-site-day grain and are aggregated rather than dropped.
- The complete machine-readable audit is stored in `artifacts/data_audit.csv`.
