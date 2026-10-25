# Core AI Engineer Capstone: From Question to Reliable Prediction

This capstone joins Python, data validation, classical machine learning, evaluation, and AI engineering. It is intentionally small enough to reproduce on a laptop. The goal is a defensible workflow, not a leaderboard score.

## The brief

Choose an openly licensed tabular dataset with a clear prediction target. Examples include predicting a future event, a category, or a numeric measurement. Before downloading, write down:

- who would use the prediction;
- when the prediction would be made;
- what action might follow;
- what a false positive and false negative mean;
- which data license permits the intended use.

Do not use sensitive personal data. If the task affects people, include a plan for human review and subgroup error analysis.

## Deliverables

Submit a repository containing:

1. **Problem card:** target, prediction time, unit of analysis, intended use, out-of-scope use.
2. **Data card:** source, license, row grain, date range, known gaps and biases.
3. **Data audit:** schema, missingness, invalid values, duplicate checks, target prevalence.
4. **Split contract:** group/time/stratification rule and leakage analysis.
5. **Baseline report:** a simple rule or constant predictor and a predeclared metric.
6. **Model comparison:** one interpretable baseline and at least two model families.
7. **Error report:** confusion matrix or residual analysis, important slices, and failure examples.
8. **Reproducible run:** configuration, dependencies, fixed random seeds where relevant, saved model, run instructions.
9. **Prediction interface:** a command-line interface or API with schema validation and clear errors.
10. **Operations note:** expected latency, memory, cost, monitoring signals, privacy/security risks, rollback condition.

## Stage 1 — Frame the task

Write the prediction contract before inspecting scores:

> At [prediction time], for each [unit], use only [available information] to estimate [target over horizon]. The output supports [decision], but must not be used for [excluded decisions].

Define the population and unit. If one person contributes many rows, say whether the goal is new rows from known people or generalization to unseen people.

**Exit check:** A reviewer can tell what one row means, when its features exist, and when its label becomes known.

## Stage 2 — Audit data before cleaning

Build a report with:

- number of rows and columns;
- key uniqueness and duplicate rates;
- type, missingness, and invalid-value counts;
- target prevalence or numeric target distribution;
- time coverage and group frequencies;
- likely label leakage and post-outcome fields.

Keep an untouched copy of the raw input. Every cleaning rule should be reproducible and justified by the data contract.

**Exit check:** You can explain which records are excluded, why, and whether exclusions change the target population.

## Stage 3 — Freeze an evaluation design

Choose the split from the deployment claim:

- random and stratified for independent exchangeable examples;
- group-separated for new-entity generalization;
- time-ordered for future prediction;
- a combination when both grouping and time matter.

Use training-only fit for imputation, scaling, feature selection, and learned encodings. Keep validation for model selection and reserve the test set for the frozen workflow.

**Exit check:** Train, validation, and test do not share forbidden groups, future information, or duplicate records.

## Stage 4 — Build baselines before complexity

At minimum, compare:

1. a constant or domain-rule baseline;
2. a regularized linear/logistic model;
3. a tree-based model such as a random forest or gradient-boosted trees.

For regression, include a meaningful absolute or squared-error metric and inspect residuals. For classification, report class counts, confusion matrix, precision/recall or task-specific costs, and probability quality if probabilities drive decisions.

Do not select the model from training score. Keep a run table with configuration, seed, training score, validation score, and notes.

## Stage 5 — Analyze failure, not only averages

Inspect examples with:

- largest residuals or highest-confidence errors;
- missing or unusual inputs;
- important groups or time windows;
- common feature ranges;
- low-confidence predictions;
- high-impact false positives and false negatives.

Describe whether each issue looks like data quality, target ambiguity, distribution shift, model capacity, or an operational constraint. Do not claim causation from feature importance.

## Stage 6 — Package and serve

Move reusable data and prediction code out of notebooks. Define an input contract with types, allowed ranges, missing-value behavior, and version. Verify that the saved model and preprocessing are loaded together.

The interface must:

- reject malformed inputs clearly;
- return a prediction plus model/version metadata;
- avoid logging secrets or raw sensitive fields;
- handle model-load or dependency errors safely;
- include a small reproducible example.

## Stage 7 — Monitor and roll back

Separate service health from model quality.

**Service signals:** request count, errors, timeouts, latency percentiles, memory, and cost.

**Data signals:** schema changes, missingness, feature ranges, population mix, and delayed labels.

**Quality signals:** task metrics, calibration when relevant, slice performance, and action volume.

State who receives alerts, what evidence triggers investigation, and how to roll back to the previous model or a safe fallback.

## Suggested report structure

1. Executive summary in plain language.
2. Problem and data contract.
3. Validation and leakage controls.
4. Baselines and model-selection procedure.
5. Results with uncertainty and slices.
6. Error examples and limitations.
7. Reproduction instructions.
8. Deployment, monitoring, security, and rollback plan.

## Rubric (100 points)

| Area | Points | Full-credit evidence |
| --- | ---: | --- |
| Problem framing and data license | 10 | Prediction unit/time, action, limits, and reuse permission are explicit. |
| Data quality and provenance | 15 | Raw data is preserved; quality issues and cleaning rules are reproducible. |
| Split and leakage controls | 20 | Split matches deployment; learned preprocessing is fitted inside training folds. |
| Baselines and model comparison | 15 | Baseline is meaningful; model comparisons use a consistent protocol. |
| Error analysis and uncertainty | 15 | Metrics match the task; failures and subgroup/time slices are discussed. |
| Software and reproducibility | 10 | Another learner can install, run, test, and reproduce the result. |
| Operations and responsible use | 15 | Interface, monitoring, privacy, security, cost, and rollback are addressed. |

## Review questions

- What would make the reported score invalid?
- What happens if the label definition changes?
- Which user group might experience worse errors?
- What happens when an input column disappears?
- How will you know the model is no longer useful?
- Can a human override the output, and is that action recorded?

## Completion standard

The capstone is complete when another learner can reproduce the baseline and chosen model from a clean checkout, verify the split, inspect known failure cases, and understand the deployment limitations without asking you to interpret undocumented choices.
