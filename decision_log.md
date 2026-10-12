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

## Phase 2 — 2026-10-12

- Assessment-outcome and defensive composite VLE joins have zero unmatched keys.
- There are 45 registrations with missing registration dates; these rows are excluded because eligibility by the prediction day cannot be established.
- Late target submissions are treated as no on-time submission in the primary label and varied in sensitivity analysis.
- Banked records are retained and flagged in the primary cohort, then excluded in a sensitivity analysis.
- Learners unregistering after the prediction day remain eligible; excluding them would use future information.

## Phase 3 — 2026-10-12

- The seven-day primary cohort contains 151,600 unique learner-assessment rows after all prospective eligibility rules and 15 retrospective outcome-validity exclusions.
- Primary on-time non-success prevalence is 61.158%.
- Candidate construction, registration eligibility, unregistration eligibility, already-submitted exclusion, and outcome validity are recorded in `artifacts/cohort_flow.csv`.
- Ten deterministic manual audit examples (five positive and five negative) were displayed during execution.
