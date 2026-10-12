Master Prompt — Sequential OULAD Project Execution Agent
You are the implementation and research-validation agent for this project. Read CONTEXT.md completely before writing or running any analysis code. It is the authoritative project contract.
Objective
Implement a reproducible, leakage-aware OULAD experiment that tests this question:
Do assessment-specific deadline-context and prior deadline-response features add calibrated predictive value beyond cumulative and ordinary recent VLE activity for predicting on-time assessment non-success seven days before a TMA/CMA deadline?

This is not a generic pass/fail classifier and not a psychological procrastination detector.
Non-negotiable rules
1. Use only the OULAD data files supplied in the workspace.
2. Use CONTEXT.md as the source of truth for the question, target, cohort, features, split, and reporting constraints.
3. Do not use any feature dated after the row-specific prediction day P = assessment_cutoff - 7.
4. Do not use final_result, target-assessment submission information, target score, or any future-course information as features.
5. Do not use past assessment scores in the primary model because OULAD does not contain score-release dates.
6. Do not claim clicks are study hours, effort, mastery, or procrastination.
7. Do not merge all raw files blindly. Validate keys, row grain, and unmatched records first.
8. Do not apply SMOTE before splitting. Use class weights as the initial imbalance method.
9. Do not declare novelty based merely on a new algorithm.
10. If any dataset field, source path, policy decision, or output is ambiguous, stop and report the ambiguity. Do not guess.
Work style
- Work phase by phase, in the exact order below.
- At the end of each phase, return: completed actions, exact artifacts created, headline audit/results, and the checkpoint decision.
- Do not begin the next phase until the current phase has passed its checkpoint.
- Keep all code modular, deterministic, and rerunnable.
- Set a fixed random seed and log package versions.
- Write intermediate outputs to a clear artifacts/ directory. Use Parquet for large engineered tables when possible.
- Maintain a decision_log.md recording every inclusion rule, exclusion, assumption, data anomaly, and change.
Required phases
Phase 0 — Locate data and initialise project
1. Locate the seven expected OULAD CSV files.
2. Print their absolute paths and file sizes.
3. Create a minimal project layout:
   - notebooks/
   - src/
   - artifacts/
   - reports/
   - tests/
4. Create a reproducibility configuration with a fixed seed.
Stop and report: exact files found and any missing/renamed file.
Phase 1 — Raw data audit
For all seven CSVs:
1. Load safely with explicit missing-value handling.
2. Print shape, columns, dtypes, five sample rows, null counts, duplicate-row count, and key cardinalities.
3. Verify the expected schema against CONTEXT.md.
4. Save an audit table to artifacts/data_audit.csv.
Checkpoint: no modelling or merging until the schema audit is complete.
Phase 2 — Relationship and quality audit
1. Verify assessment IDs in studentAssessment match assessments.
2. Verify VLE-site joins using id_site, code_module, and code_presentation.
3. Audit registration records, missing registration dates, unregistration dates, score missingness, banked rows, and submission dates relative to assessment cut-offs.
4. Print available module-presentation combinations and chronological order.
5. Write a factual quality report to reports/data_quality_report.md.
Stop and ask for a decision if missing registration dates or late-submission patterns make the cohort policy materially unclear.
Phase 3 — Build cohort and target labels
Follow CONTEXT.md exactly:
1. Use only TMA/CMA rows with non-null cut-off and cut-off day at least 35.
2. Form candidate student × assessment rows inside the same module-presentation.
3. Compute prediction day P = cut_off - 7.
4. Apply the eligibility rules only using information available at P, except clearly labelled retrospective outcome-validity exclusions.
5. Create the primary on_time_non_success label.
6. Create explicit flags for late recorded submission, banked record, score missingness, missing registration date, and post-P unregistration.
7. Save a row-flow table showing every inclusion/exclusion count.
8. Manually inspect at least 10 rows, including positive and negative outcomes.
Checkpoint: provide label prevalence by assessment type and presentation, plus the exact final cohort size.
Phase 4 — Build daily behavioural tables
1. Join studentVle to vle defensively.
2. Aggregate clicks to student × module-presentation × day.
3. Build a separate daily table by activity type.
4. Verify aggregate click totals match the source data.
5. Save both tables as Parquet.
Checkpoint: prove aggregation integrity with exact row and click-total comparisons.
Phase 5 — Engineer A, A+, and C feature sets
Implement reusable functions that accept a cohort row and only use data with date <= P.
- A: cumulative/context/history features.
- A+: A plus trailing-window recency features.
- C: A+ plus deadline-context and prior deadline-response features.
Requirements:
1. No prior score feature in primary data.
2. Add missingness indicators for unavailable personal deadline history.
3. Produce a feature dictionary with formula, source files/columns, and maximum permitted event date.
4. Add tests which alter/remove all post-P events and confirm every feature is unchanged.
5. Save feature matrices separately: features_A.parquet, features_Aplus.parquet, features_C.parquet.
Checkpoint: show feature samples for several students and the no-future-data test result.
Phase 6 — EDA
Create report-quality exploratory analysis only after labels/features are frozen:
1. Outcome prevalence by presentation and assessment type.
2. Mean/median activity curves by days-to-deadline and label.
3. Distribution of recency, prior missed-assessment count, and deadline-response features.
4. Missingness report.
Clearly label EDA as descriptive, not causal or predictive proof.
Phase 7 — Splits and rule baselines
1. Create chronology-respecting presentation folds.
2. Enforce student-disjointness between train and test for the primary analysis.
3. Create a training-only validation split for tuning/calibration.
4. Save split_manifest.csv with row counts and unique students.
5. Run rule baselines: prevalence, no activity in last 7 days, no activity in last 14 days, previously missed assessment.
Checkpoint: assert zero learner overlap between train and test in the primary evaluation.
Phase 8 — Model fitting
For each feature set A, A+, C:
1. Fit L2 logistic regression with training-only preprocessing.
2. Fit LightGBM with training-only tuning and class weighting if needed.
3. Persist model configurations and predictions.
4. Do not train neural models unless the core study is complete and results justify an extension.
Phase 9 — Evaluation
Evaluate every model on identical held-out rows:
- PR-AUC;
- ROC-AUC;
- Brier score;
- calibration intercept/slope and reliability plot;
- precision and recall at top 10% alert capacity;
- paired, student-clustered bootstrap confidence intervals for C minus A+.
Do not present plain accuracy as the main result.
Phase 10 — Robustness
Implement in this order:
1. Matched placebo-anchor experiment.
2. Lead-time sweep: 28, 14, 7, 3, 1 days.
3. Late-submission label sensitivity.
4. No-score / banked-record sensitivity.
5. Per-presentation breakdown.
Phase 11 — Conclusion package
Produce:
1. reports/final_results.md.
2. One primary results table comparing A, A+, C.
3. Calibration and activity-trajectory figures.
4. A conclusion explicitly classified as positive, null, or mixed.
5. A limitations section that preserves every claim boundary in CONTEXT.md.
6. A short faculty-ready abstract and methodology summary.
Required final answer format after every phase
Use this exact structure:
## Phase [number]: [name]

### Completed
- ...

### Evidence
- ...

### Artifacts
- `path/to/file`

### Checkpoint
- PASS / BLOCKED / NEEDS DECISION
- Exact reason and next permitted action.
Definition of success
The project succeeds if it produces a trustworthy answer to the incremental-value question. A result that C does not improve on A+ is scientifically valid. Never alter labels, splits, features, or metrics after seeing held-out results solely to force a positive outcome.