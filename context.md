OULAD Deadline-Anchored Early Warning — Project Context and Execution Contract
1. Project identity
Assigned problem statement: Last-Minute Learning Behaviour Detection
Dataset: Open University Learning Analytics Dataset (OULAD) only
Project type: Supervised machine-learning study with a leakage-aware, assessment-level experimental design.
Working title
Do Learners’ Prior Deadline-Response Patterns Improve Seven-Day-Ahead Prediction of On-Time Assessment Non-Success Beyond Recent VLE Activity? Evidence from OULAD
Short title
Deadline-Anchored Assessment Early Warning on OULAD
2. The actual problem being solved
This project does not claim to detect procrastination as a psychological trait. OULAD does not record a learner’s intention, offline study time, stress, motivation, or actual time spent studying.
It studies an observable and operationally useful question:
Given a known TMA/CMA assessment deadline, can a learner’s recorded VLE engagement and prior deadline-response history identify, seven days beforehand, whether that learner is at risk of on-time assessment non-success?

The target is assessment-level rather than final-course-level. A tutor needs to know whether a learner is likely to struggle with the next specific assessment, not merely whether they may eventually fail a course.
3. Research question and hypotheses
Primary research question
Do assessment-specific deadline-context and prior deadline-response features provide incremental, calibrated predictive value beyond cumulative and recent VLE activity for predicting on-time assessment non-success seven days before a TMA/CMA deadline?

Null hypothesis, H0
After accounting for cumulative activity and ordinary recent-activity features, deadline-context and prior deadline-response features do not improve out-of-sample prediction or calibration.
Alternative hypothesis, H1
Deadline-context and prior deadline-response features improve out-of-sample prediction and/or calibration beyond cumulative and ordinary recent activity.
What a valid conclusion looks like
- Positive result: the deadline-context feature set improves PR-AUC and/or calibration over the strict recent-activity baseline, on the same held-out rows; the paired confidence interval excludes zero; the gain is not reproduced by a matched placebo anchor.
- Null result: recent activity explains the predictive signal; deadline-specific context adds no reliable incremental value. This is still a valid and publishable project conclusion.
- Mixed result: the feature set improves ranking but worsens calibration, or helps only certain presentations. Report that limitation directly.
4. OULAD data contract
Files used
File	Role in this project	Important columns
assessments.csv	assessment schedule and context	code_module, code_presentation, id_assessment, assessment_type, date, weight
studentAssessment.csv	recorded submission outcome	id_assessment, id_student, date_submitted, is_banked, score
studentVle.csv	daily online interaction behaviour	code_module, code_presentation, id_student, id_site, date, sum_click
vle.csv	maps VLE sites to activity types	id_site, code_module, code_presentation, activity_type, week_from, week_to
studentInfo.csv	course registration context and final result	code_module, code_presentation, id_student, num_of_prev_attempts, studied_credits, etc.
studentRegistration.csv	registration/unregistration timing	code_module, code_presentation, id_student, date_registration, date_unregistration
courses.csv	presentation duration	code_module, code_presentation, module_presentation_length


Definitions that must remain accurate
- OULAD date values are days relative to the start of a module presentation.
- sum_click is a count of recorded VLE clicks. It is not study duration, effort, learning gain, or time-on-task.
- studentVle is one student × VLE site × day, not one learner-day.
- studentAssessment provides a recorded submission date and score, but not a complete history of multiple attempts or score-release time.
- Join studentVle to vle on id_site, code_module, and code_presentation for defensive correctness.
Claims this dataset cannot support
Never claim any of the following:
- psychological procrastination diagnosis;
- actual offline study hours;
- topic-level mastery or knowledge tracing;
- causal effects of reminders, interventions, or recommendations;
- the best allocation of study minutes across topics;
- whether a learner saw a score before the next prediction time;
- why a learner did not submit.
5. Unit of analysis, prediction point, and label
Unit of analysis
One row represents one student × upcoming non-exam assessment.
Use only TMA and CMA assessments. Do not use Exams as the primary task because individual final-exam outcomes are incomplete or inconsistently available in OULAD.
Prediction point
For assessment a with official cut-off day c_a:
\[
P_{s,a} = c_a - 7
\]
The model may use only information whose event date is less than or equal to P_{s,a}.
Recommended primary cohort
Keep a row only when all conditions are true:
1. assessment_type is TMA or CMA.
2. Assessment cut-off date is non-null and c_a >= 35. This ensures a 28-day history can exist without manufacturing pre-course inactivity.
3. The learner is registered by P_{s,a}. Audit missing registration dates before choosing the exact policy.
4. The learner has not unregistered on or before P_{s,a}.
5. The learner has not already recorded a submission for the target assessment on or before P_{s,a}.
Primary outcome
Call the target on-time assessment non-success, not generic failure.
\[
Y_{s,a} = 1
\]
if either:
- no recorded submission exists on or before the official cut-off c_a; or
- an on-time recorded submission exists and its recorded score is below 40.
Otherwise:
\[
Y_{s,a} = 0
\]
Required label sensitivity analyses
1. Treat any late recorded submission as a submission rather than non-submission.
2. Exclude banked assessments, but disclose that is_banked is outcome-side metadata and not a prediction-time feature.
3. Run a no-score target: on-time submission versus no on-time submission.
Learners who unregister after P_{s,a} remain in the primary cohort. Removing them would use future information and artificially remove genuine non-submitters.
6. Contribution and novelty boundaries
Defensible contribution
The project tests whether assessment-specific deadline context and a learner’s own observed history around earlier deadlines improve seven-day-ahead prediction beyond what cumulative and ordinary recency features already provide.
What is not novel
- predicting student performance from OULAD;
- predicting course-level failure/withdrawal;
- using LSTM, gradient boosting, random forest, logistic regression, SHAP, or click counts;
- using time-varying learning data in general.
Careful gap statement
Use this wording in reports:
Earlier OULAD research has modelled course-level outcomes, engagement, at-risk learners, and in at least one case deadline-aligned first-assessment submission. This study evaluates whether deadline-context and prior deadline-response features add incremental value beyond strong cumulative and recent-activity baselines across TMA/CMA assessments, with temporal validation and calibration analysis.

Never write “this has never been explored” unless a further full literature review proves it.
7. Feature sets
All feature engineering must be computed using only data available on or before the row-specific prediction day P.
A — Cumulative baseline
- target assessment type and weight;
- module and presentation identifiers;
- registration timing and available exposure days;
- number of previous attempts and studied credits;
- cumulative clicks, active days, distinct sites, and activity types through P;
- observed prior submission history only: count of prior assessments, on-time submission rate, missed-prior-assessment count.
Do not include prior scores in the primary model. OULAD does not record score-release dates. Prior scores may be evaluated only as an explicitly assumption-based sensitivity model.
A+ — Strict ordinary-recency baseline
A plus:
- clicks in trailing 1, 3, 7, 14, and 28-day windows ending at P;
- active days in those windows;
- recency: days since last VLE activity;
- distinct sites/activity types in the trailing windows.
This is an essential baseline. If later feature sets cannot beat A+, their apparent gain is probably ordinary recency rather than deadline relevance.
C — Deadline-context and prior deadline-response set
A+ plus:
- days since the previous assessment cut-off in the same presentation;
- number and total weight of other assessment cut-offs in the next 7/14 days, excluding the target assessment;
- prior on-time submission rate;
- proportion of past assessments submitted exactly on their cut-off day;
- average prior deadline surge: activity in the last 3 days before earlier cut-offs divided by activity in days 4–28 before them;
- change in activity type mix in the most recent 14 days versus the preceding 14 days;
- optional behavioural shape features: slope, collapse flag, and concentration of clicks.
Use an explicit missingness indicator for personal-history features where a learner has no earlier eligible assessment.
Secondary exploratory features
- OLS slope of log(1 + daily_clicks) over the last 14/28 days;
- collapse flag: no clicks in the last seven days after non-zero activity in the preceding 21 days;
- top-day share: largest single-day click total divided by 28-day click total;
- Jensen-Shannon divergence in activity-type shares across the two 14-day windows.
Treat these as exploratory unless they are pre-registered before looking at held-out outcomes.
8. Anti-leakage rules
These rules are mandatory.
1. No VLE event after P may affect any feature.
2. No target assessment submission outcome may be used as a feature.
3. Do not use final_result; it occurs after the course.
4. Do not use target-assessment score or submission date in features.
5. Do not use prior scores in the primary model because their availability date is unknown.
6. When forming training/test sets, prevent the same id_student appearing in both sides for the primary generalisation result. Report a secondary temporal-only result separately if retaining repeat learners is necessary.
7. Fit encoders, scalers, imputers, feature selection, resampling, calibration, and hyperparameter tuning on training data only.
8. Do not apply SMOTE before splitting. Prefer class weighting first.
9. Test feature leakage: perturb or remove all post-P VLE events and assert engineered features remain unchanged.
10. Generate the target only after feature-cutoff filtering. Labels may use later outcome data; features may not.
9. Experimental design
Main comparisons
Compare the same models on the same held-out rows:
1. A: cumulative baseline.
2. A+: strict recent-activity baseline.
3. C: A+ plus deadline-context and prior deadline-response features.
Models
1. L2-regularised logistic regression: transparent baseline.
2. LightGBM: nonlinear tabular benchmark.
Optional only after the above work correctly:
- SHAP explanations for the final LightGBM model;
- Platt scaling if a validation set shows poor calibration;
- a hierarchical Bayesian logistic model as a separate advanced extension.
Do not start with LSTM, Transformer, GNN, RL, or Bayesian modelling. They are not the contribution and can make leakage/debugging harder.
Split strategy
Primary goal: evaluate future-presentation generalisation while avoiding student identity leakage.
1. Sort presentations chronologically.
2. Train only on earlier presentations.
3. Test on later presentations.
4. Remove from a test fold any learner whose id_student occurs in its training data for the student-disjoint primary analysis.
5. Use grouped cross-validation by id_student within the training period for tuning.
6. Hold out a time-respecting validation portion from training for calibration and model selection.
Before committing to folds, print the exact available module-presentation combinations and row counts. Some presentations/modules may be absent in particular terms.
Evaluation
Primary metrics:
- PR-AUC;
- ROC-AUC;
- Brier score;
- calibration intercept/slope and reliability plot;
- Precision@top 10% and Recall@top 10%.
Uncertainty:
- paired bootstrap confidence intervals clustered by id_student, comparing C against A+ on identical test rows.
Required robustness checks
1. Placebo anchor: compute matched features at pseudo-deadlines selected to resemble course age but not a real assessment cut-off. If it gives the same gain as C, the signal is generic recency rather than deadline context.
2. Lead-time sweep: repeat at 28, 14, 7, 3, and 1 days before the deadline.
3. No-score sensitivity: avoid score-based outcome or feature assumptions.
4. Late-submission sensitivity: vary treatment of late submissions.
5. Banked-record sensitivity.
6. Per-presentation results: do not hide weak generalisation behind pooled averages.
10. Sequential notebook execution plan
The notebook must be executed in order. Never jump to modelling before the earlier audit/checkpoint is complete.
Phase 0 — Environment and reproducibility
Cells: imports, paths, fixed random seed, version logging, output directories.
Output: reproducibility configuration.
Phase 1 — Raw data audit
Cells: load all seven CSV files; shapes; columns; dtypes; head; null table; duplicate table; key uniqueness; date ranges.
Output: data_audit.csv and a concise audit table.
Checkpoint: verify all filenames and schemas before any merge.
Phase 2 — Relational integrity audit
Cells: validate joins; inspect unmatched IDs; inspect assessment types; modules/presentations; registration missingness; score and date-submitted distributions; banked records; date-submitted relative to cut-off.
Output: join coverage and data-quality report.
Checkpoint: explicitly choose policies for missing registration dates, banked rows, and late submissions.
Phase 3 — Cohort and labels
Cells: filter TMA/CMA; build student × assessment candidate rows; calculate row-specific P = cutoff - 7; apply eligibility rules; attach labels; generate exclusion flow table.
Output: cohort_rows.parquet, label prevalence by presentation/type, row-flow chart/table.
Checkpoint: manually inspect at least 10 rows, including positive and negative labels.
Phase 4 — Daily behavioural base table
Cells: defensively join VLE metadata; aggregate to student × presentation × day; build activity-type daily totals; check event coverage.
Output: daily_activity.parquet and daily_activity_by_type.parquet.
Checkpoint: test that daily aggregates equal the original click totals.
Phase 5 — Feature engineering
Cells: build A, A+, C features; add missingness flags; define feature dictionary; run no-future-data unit test.
Output: separate feature tables and feature dictionary.
Checkpoint: choose a few cohort rows and prove every feature only uses events dated <= P.
Phase 6 — EDA before modelling
Cells: cohort balance; mean/median activity curves in days to deadline; activity by outcome; prior-deadline behaviour; correlation/missingness plots.
Output: labelled exploratory figures.
Checkpoint: EDA must not alter the pre-defined primary target or evaluation protocol.
Phase 7 — Splits and simple rules
Cells: list presentation chronology; make train/validation/test folds; enforce student disjointness; train prevalence/no-activity/missed-prior rules.
Output: split manifest and rule-baseline metrics.
Checkpoint: validate no id_student overlap between train and test in primary evaluation.
Phase 8 — Models
Cells: logistic regression pipeline and LightGBM pipeline for A, A+, C; tune only in training; fit final models.
Output: out-of-fold validation table and frozen model configurations.
Phase 9 — Evaluation and calibration
Cells: discrimination metrics; calibration; precision@capacity; paired clustered bootstrap; compare deltas C−A+.
Output: final metrics table, confidence intervals, calibration/reliability figures.
Phase 10 — Robustness and interpretation
Cells: placebo test; lead sweep; sensitivity analyses; per-presentation results; optional SHAP.
Output: robustness table and result narrative.
Phase 11 — Final report material
Cells: figures/table export; methodology text; limitations; reproducibility instructions.
Output: report-ready findings package.
11. Minimal viable execution order
If time is limited, complete these in order:
1. Phases 0–4 completely.
2. A, A+, C feature sets only.
3. Logistic regression and LightGBM only.
4. One temporal, student-disjoint held-out test.
5. PR-AUC, Brier score, calibration plot, and A+ versus C comparison.
6. One placebo test.
Do not sacrifice leakage control for a larger model list.
12. Agent operating instructions
The implementation agent must:
- execute one phase at a time and stop at each checkpoint;
- show concise evidence from outputs before proceeding;
- never silently modify the research question, target, or cohort rules;
- not manufacture unavailable OULAD fields;
- not use any post-prediction data in features;
- write reusable functions rather than duplicating feature logic in notebook cells;
- save intermediate tables with schema/version metadata;
- record every design decision, row exclusion, and assumption;
- report an error or ambiguity instead of guessing;
- treat a null result as scientifically valid.
13. Final reporting language
Allowed claim
We evaluated whether observed VLE engagement and prior deadline-response patterns improve seven-day-ahead prediction of on-time assessment non-success for OULAD TMA/CMA assessments beyond cumulative and recent activity baselines.

Not allowed
We detected procrastinators.

We measured study effort or study time.

We proved deadlines caused behaviour.

We identified the optimal intervention or study plan.