# OULAD data quality report

Generated from the supplied ZIP.

## Audit counts

- unmatched_studentAssessment_ids: 0
- unmatched_studentVle_composite_keys: 0
- duplicate_vle_metadata_keys: 0
- duplicate_registration_keys: 0
- registration_date_missing: 45
- unregistration_date_missing: 22,521
- score_missing: 173
- banked_rows: 1,909
- late_submission_rows: 49,318
- student_assessment_duplicate_keys: 0

## Fixed policies

- **missing_registration_date** — exclude: eligibility by P cannot be established
- **banked_target_record** — retain and flag in primary; exclude in sensitivity
- **late_target_submission** — primary: no on-time submission; sensitivity: any recorded submission counts
- **post_prediction_unregistration** — retain: removing it would use future information

No rows were silently repaired. Composite VLE joins and assessment joins passed coverage assertions.