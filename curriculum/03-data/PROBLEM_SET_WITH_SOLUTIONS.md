# Data Engineering and Validation: Worked Problem Set

Write assumptions before reading the worked answers. Small examples make the reasoning visible; production systems also need contracts, monitoring, and access controls.

## 1. Join cardinality

An orders table has 100 rows, one per order. A line-items table has 240 rows. You join by order_id. How many rows should an inner join have? What could cause more than 240?

### Solution

At most 240 if every item matches exactly one order; fewer if some items have no matching order. More than 240 means an order_id is duplicated in orders or the join condition is incomplete. Verify key uniqueness and row grain before joining.

## 2. Aggregation after a one-to-many join

Order A is worth 30 and has three line items. Order B is worth 20 and has one item. What happens if you sum order value after joining items to orders?

### Solution

The joined sum is 30+30+30+20=110, because order A is repeated. The correct total is 50. Aggregate at order grain before the join, or sum genuine line-level amounts.

## 3. Point-in-time correctness

At 09:00 a model predicts default in the next 30 days. A repayment occurs at 10:00 and the source status changes to “repaid” at 10:05. Can a feature for the 09:00 prediction use that status?

### Solution

No. It was not available when the prediction was made. Track event time, source availability time, and prediction time. Enforce availability_time <= prediction_time, including for later backfills.

## 4. Split by entity

A dataset has 10 images for each of 100 patients. Why can random image-level splitting inflate validation performance?

### Solution

Images from one patient share appearance and acquisition patterns. Split by patient ID so a patient appears in only one partition. Add a time or site holdout if deployment targets future data or a new hospital.

## 5. Fit preprocessing on training data

Training values are [1,3,5] and validation has [100]. You standardize using all four values. What is wrong?

### Solution

Validation influenced the fitted transform. Fit mean and standard deviation on train only; mean is 3 and population standard deviation is sqrt(8/3). Freeze those values for validation, test, and serving.

## 6. Missingness can carry information

Sensor readings are missing specifically when a machine overheats. Is a missingness indicator useful? What can go wrong with median imputation alone?

### Solution

Missingness may itself indicate overheating, so an indicator can preserve useful information. Verify that the signal is available at prediction time and not caused by post-outcome logging. Median imputation alone can hide the pattern.

## 7. Deterministic deduplication

For each entity, keep the greatest event_time; break ties using greatest ingestion_id. Describe a deterministic method.

### Solution

Partition by entity, sort by event_time descending and ingestion_id descending, then retain row number 1. Never rely on file order. Preserve duplicate records for audit when retries are expected.

## 8. Event time and late arrivals

A stream uses 10-minute event-time windows. At processing time 12:20, it receives an event with event_time 12:02. Which window owns it? What does a watermark do?

### Solution

It belongs to 12:00–12:10. A watermark estimates that events before a point are unlikely to arrive and allows the system to finalize windows. A lateness policy decides whether late events are dropped, revise results, or go through correction.

## 9. Idempotent retry

A daily job adds transactions to a cumulative table, crashes after writing, then retries. How do you avoid double counting?

### Solution

Use a stable batch key and atomic upsert/merge, or replace an immutable output partition on retry. The same input should yield the same committed result. A blind append is not idempotent.

## 10. Feature freshness

Serving requires features no older than 20 minutes. A late job writes values based on data updated 40 minutes ago. What should happen?

### Solution

Timestamp features and validate age when used. For stale data, follow an explicit policy: recompute, use a safe fallback, abstain, or return a controlled error. Alert on freshness violations; never silently treat a stale value as current.

## 11. Quality checks

Age must be within [0,120]. Maximum observed age is 87 and there are no nulls. Does this prove the data is correct?

### Solution

No. Values can be duplicated, shifted, mislabeled, stale, or semantically invalid while in range. Add uniqueness, distribution, referential-integrity, freshness, and domain checks; inspect samples and lineage.

## 12. Distribution shift

A feature's population stability index is 0.32 after a product launch. Does that prove model accuracy fell?

### Solution

No. PSI indicates a distribution change under the chosen binning, not predictive quality or causality. Inspect source changes, slices, and delayed ground truth. A threshold is a heuristic, not a universal law.

## 13. SQL ranking

For each user, rank events newest first. How do ROW_NUMBER and RANK differ when two events share the newest timestamp?

### Solution

ROW_NUMBER assigns unique positions; add a deterministic tie-breaker if exactly one row is needed. RANK gives tied rows the same rank and leaves a gap. DENSE_RANK ties without a gap.

## 14. Define metric grain

Give two valid meanings for “average purchase amount.”

### Solution

Average per transaction weights purchases equally. Average per customer first calculates each customer's average, then weights customers equally. Specify the unit, refund/cancellation policy, and time window.

## Review checklist

- Can you state each table's grain and key?
- Can you reason about join cardinality and duplicates?
- Can you show every feature was available at prediction time?
- Are splits and preprocessing done at the correct unit and time?
- Are freshness, retries, late events, and corrections explicit?
- Do quality checks cover semantics as well as nulls and ranges?
