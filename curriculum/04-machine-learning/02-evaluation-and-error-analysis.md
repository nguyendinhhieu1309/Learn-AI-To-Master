# Evaluation and Error Analysis

## Learning objectives

Choose evaluation metrics that match a task, use data splits correctly, and inspect how and where a model fails.

## 1. Classification metrics

For a binary classifier:

- **True positive (TP):** positive case predicted positive.
- **False positive (FP):** negative case predicted positive.
- **False negative (FN):** positive case predicted negative.
- **True negative (TN):** negative case predicted negative.

The core metrics are

$$
\operatorname{Precision}=\frac{TP}{TP+FP},
\qquad
\operatorname{Recall}=\frac{TP}{TP+FN},
\qquad
F_1=2\frac{\operatorname{Precision}\cdot\operatorname{Recall}}
{\operatorname{Precision}+\operatorname{Recall}}.
$$

Precision answers: “Of predicted positives, how many were correct?” Recall answers: “Of actual positives, how many did we find?” $F_1$ balances the two, but can hide the costs of different error types. Accuracy is

$$
\operatorname{Accuracy}=\frac{TP+TN}{TP+TN+FP+FN}.
$$

It can look high when one class dominates.

ROC-AUC measures ranking across thresholds; precision-recall curves are often more informative with rare positives. AUC does not say which threshold to deploy.

## 2. Regression metrics

For numerical predictions $\hat y_i$ and targets $y_i$:

$$
\operatorname{MAE}=\frac{1}{n}\sum_{i=1}^{n}|y_i-\hat y_i|,
\qquad
\operatorname{MSE}=\frac{1}{n}\sum_{i=1}^{n}(y_i-\hat y_i)^2,
\qquad
\operatorname{RMSE}=\sqrt{\operatorname{MSE}}.
$$

MAE is in target units and is less dominated by large errors. MSE emphasizes large errors and has squared units. RMSE has the same units as the target and still emphasizes large errors.

Always compare against a baseline and inspect residuals by range or relevant groups. A good average error can mask failures on a subgroup.

## 3. Thresholds and costs

Classification models may produce scores or probabilities. The threshold controls the precision/recall trade-off. Select it on validation data based on the real cost of false positives and false negatives. At 0.5, many valid decisions are missed if classes or costs are asymmetric.

## 4. Split strategy and cross-validation

Use a test set once for a final estimate after choosing the model. Cross-validation repeats train/validation splits to compare settings more robustly on limited data. Keep groups and time ordering intact where needed. If labels are imbalanced, stratify when that matches deployment.

## 5. Error analysis

Do not stop at a score. Inspect false positives and false negatives, residual patterns, confidence, data quality, and performance by meaningful subgroup. Label error categories: ambiguous labels, missing context, rare patterns, distribution shift, or a model limitation. Use these categories to propose the next experiment.

## Worked example

An alarm classifier has 20 true alarms and 80 non-alarms. It catches 15 real alarms but also raises 10 false alarms. Then

$$
\operatorname{Precision}=\frac{15}{15+10}=0.60,
\qquad
\operatorname{Recall}=\frac{15}{15+5}=0.75,
\qquad
\operatorname{Accuracy}=\frac{15+70}{100}=0.85.
$$

If missed alarms are costly, the 0.75 recall may matter more than accuracy. If alarms are expensive to investigate, false positives matter too.

## Practice with answer

If $TP=8$, $FP=2$, and $FN=4$, precision is $8/10=0.8$, recall is $8/12\approx0.67$, and $F_1\approx0.73$. Explain which change to the decision threshold might increase recall and what cost it could add. Lowering the threshold often catches more positives but also creates more false positives.

## Common mistakes

- Reporting a single metric without counts, threshold, or comparison.
- Calculating metrics on training data and calling them generalization.
- Repeatedly consulting the test set while tuning.
- Averaging away uneven performance across groups or operating conditions.
