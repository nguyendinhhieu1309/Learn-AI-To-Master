# Data Validation and Leakage: A Worked Workshop

This workshop focuses on a difficult question: **what information was actually available when the prediction would have been made?** Correct splitting and point-in-time features matter more than a sophisticated model.

## Learning objectives

- Choose a split that matches deployment.
- Separate event time, feature availability time, prediction time, and label time.
- Detect common forms of target leakage and group leakage.
- Explain why preprocessing must be fitted using training data only.

## 1. Define the prediction moment

Suppose a service predicts whether an order will be delivered late. A row represents an order at the moment it is placed, time $t_0$.

| Field | Meaning at prediction time |
| --- | --- |
| `order_id` | unique order identifier |
| `customer_id` | customer who placed it |
| `order_time` | timestamp of order |
| `promised_days` | delivery promise known at order time |
| `warehouse_id` | assigned warehouse if assignment is already made |
| `delivered_time` | observed only after delivery |
| `late_label` | derived after delivery |

`delivered_time` is not a valid feature at $t_0$. A dataset can be perfectly clean and still be invalid for prediction if it includes information created after the prediction moment.

Before building features, write a one-sentence prediction contract:

> For each order, at order placement time, predict whether it will miss its promised delivery date using information available by that time.

## 2. Four times that must not be confused

- **Event time:** when something happened in the real world.
- **Availability time:** when the system could first use the record.
- **Prediction time:** when the model must produce a score.
- **Label time:** when the outcome becomes knowable.

For example, a payment event may have happened at 10:00 but arrived in the warehouse at 10:20. A prediction at 10:10 cannot use it, even though its event timestamp is earlier than prediction time.

For every feature $x_j$, require:

$$
\operatorname{available\_at}(x_j)\leq t_{\mathrm{prediction}}.
$$

For a label built over a future window $[t_0,t_0+H]$, do not include rows whose label window has not matured by the dataset cutoff.

## 3. Choose the split from the data-generating process

### Independent, exchangeable examples

A random split can be reasonable when examples are independent and future examples are drawn from the same process. Stratification can preserve approximate class proportions; it does not repair dependence or time leakage.

### Repeated people, devices, or locations

If the same customer appears in train and test, a model may memorize customer-specific patterns. Use group separation when deployment requires generalization to unseen customers.

### Future prediction

If the model will predict future observations, train on earlier periods and test on a later period. Keep a temporal gap when labels or features overlap across the boundary.

### A split is part of the claim

“The model has 92% accuracy” is incomplete without the sampling unit and split rule. A random row split, a new-customer split, and a future-time split answer different questions.

## 4. Preprocessing must be learned inside each training fold

Suppose you impute a missing value with the column mean. If you calculate that mean using train and validation together, information from validation affects the learned transformation. The effect may be small, but the procedure no longer simulates a genuinely unseen validation set.

Correct sequence:

1. Define the train/validation/test split.
2. Fit imputation, scaling, feature selection, and encoding on the training partition.
3. Apply those fitted transformations to validation or test data.
4. Tune using validation only.
5. After decisions are frozen, refit the chosen pipeline on train plus validation if that matches the final protocol.
6. Evaluate the test set once.

Any choice informed by validation scores—including feature engineering and threshold selection—is model selection and belongs before the final test evaluation.

## 5. Find leakage by asking how each value was created

For each candidate feature, answer:

1. Who or what generated it?
2. At what time did it become available?
3. Could production compute it for every request?
4. Is it a direct or indirect copy of the target?
5. Does it contain an identifier or post-outcome workflow state?

Common leakage patterns:

- a status field updated only after a case is resolved;
- a feature computed from the complete dataset before splitting;
- duplicates or near-duplicates crossing partitions;
- target encoding calculated using held-out labels;
- future events included in a rolling aggregate;
- label maturity not respected at the cutoff;
- the same patient, device, household, or robot session in both train and test.

## 6. Worked rolling-feature example

For customer $c$ and order time $t$, consider “number of late deliveries in the previous 30 days.” A correct feature uses only earlier orders whose outcomes were already known:

$$
x(c,t)=\sum_{i:\;c_i=c,\;t-30d\leq t_i<t,\;\ell_i\text{ known by }t}\ell_i.
$$

The conditions matter:

- $t_i<t$ prevents including the current order or a future order.
- The 30-day window limits the history.
- “Known by $t$” prevents using a delivery outcome that had not yet occurred or arrived.

A naïve group-by on all rows can violate all three conditions.

## 7. Quality checks before model training

Record at least:

- row count, key uniqueness, and the row grain;
- null and invalid-value counts by column;
- target prevalence by time and group;
- duplicate and near-duplicate counts;
- minimum and maximum timestamps;
- label maturity and feature availability rules;
- split sizes and overlap of IDs across partitions.

Do not automatically delete outliers. First distinguish impossible values, rare valid events, and measurement artifacts.

## Exercises

1. A fraud model predicts at transaction authorization time. The table contains chargeback status, merchant category, and customer spend in the next month. Which fields can be used?
2. Ten images per patient are labeled; the goal is performance on new patients. What must be grouped during splitting?
3. A standardizer is fitted on the full data before cross-validation. Why is this wrong? Does it always make the score much larger?
4. You predict next-week demand with a feature “mean sales per store.” What timestamp rules and split do you need?
5. An event happened before a score but was ingested after the score. Is it available to the prediction?

## Answer checks

1. Merchant category is usable if available at authorization. Chargeback status and future-month spend are post-outcome information and are invalid. Verify when each field is populated, not just its name.
2. Group by patient so that all images for one patient stay in a single partition.
3. Validation statistics influence the fitted transformation. This is leakage in the evaluation procedure even if the effect on the final score happens to be small.
4. Use a time-based split. Compute the mean only from sales available before each prediction cutoff; define whether the mean is historical, rolling, or a training-period aggregate.
5. No. Availability time, not event time alone, determines whether the prediction system could use it.

## Mini-project

Take any tabular dataset and create a one-page **data and split contract** containing:

1. prediction unit and prediction time;
2. target definition and label-maturity window;
3. feature availability table;
4. split rule and reason;
5. duplicate/group-overlap checks;
6. preprocessing steps fitted only on training data;
7. known sources of bias or missingness.

Do not start model comparison until another learner can review and challenge this contract.

## Common mistakes

- Treating random splitting as a universal default.
- Comparing records from the same entity across partitions without stating the deployment objective.
- Using event time while ignoring ingestion delay.
- Calling a variable “metadata” and assuming it cannot leak.
- Reusing the test set to select a feature, threshold, or model.
