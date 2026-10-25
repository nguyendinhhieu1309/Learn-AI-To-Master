# Generalization, Validation, and Model Selection

Training a model is easy compared with measuring whether it will work on new data. This lesson explains the evaluation process and the common ways it becomes misleading.

## 1. Generalization and the data split

Let $R(f)$ be the expected loss of model $f$ on the real deployment distribution, and let $\hat{R}(f)$ be the average loss on a finite dataset. We can measure $\hat{R}$; we care about $R$. The difference is generalization error.

Use three logical roles:

- **Training set**: fit parameters.
- **Validation set**: choose features, hyperparameters, thresholds, and checkpoints.
- **Test set**: estimate final performance after decisions are fixed.

If test results guide another decision, the test set has become validation data. Keep a new untouched test set for the final claim.

Choose a split that reflects deployment:

- Random split for independent, identically distributed examples.
- Group split when records share a person, device, company, video, or source.
- Time split when future prediction is the goal.
- Spatial/site split when the model must transfer to new locations or institutions.

## 2. Leakage: the hidden shortcut

Leakage occurs when training or model selection uses information unavailable at prediction time. It may be a direct target copy, a post-outcome feature, duplicate records across splits, preprocessing fitted on all data, or a time-dependent feature that reveals the future.

Fit every learned transformation on training data only:

```python
pipeline = Pipeline([
    ("impute", SimpleImputer(strategy="median")),
    ("scale", StandardScaler()),
    ("model", LogisticRegression()),
])
pipeline.fit(X_train, y_train)
pred = pipeline.predict(X_valid)
```

The pipeline fits imputation and scaling inside each training fold. Computing global means before cross-validation leaks information from validation folds into training.

Leakage audit questions:

1. Could this feature exist at the exact time the prediction is made?
2. Can one person, object, or near-duplicate appear in both train and test?
3. Were labels or aggregate statistics computed using future outcomes?
4. Did human review, prompt tuning, or threshold selection use the test set?
5. Does the test set represent the deployment population and time period?

## 3. Cross-validation

In $k$-fold cross-validation, divide training data into $k$ folds, fit on $k-1$ folds, and evaluate on the remaining fold, repeating so each fold serves as validation once. Average fold scores and report their variation, while recognizing that folds are not independent experiments.

Use stratified folds for classification when appropriate, group-aware folds for correlated rows, and time-aware validation for forecasting. The split mechanism is part of the experiment and should be recorded.

Nested cross-validation separates hyperparameter selection from performance estimation: the inner folds tune the model; the outer folds estimate the entire selection procedure. It costs more but avoids optimistic reporting when data is limited.

## 4. Bias, variance, and learning curves

High training error suggests underfitting: the model or features cannot represent the pattern, optimization is incomplete, or regularization is too strong. Low training error with much worse validation error suggests overfitting or a distribution/split mismatch.

Plot learning curves over increasing training-set size. If both train and validation errors remain high and close, more data of the same type may not help much. If train error is low while validation is high, more representative data, stronger regularization, or simpler modeling may help.

These diagnoses are clues, not proofs. First rule out bugs, leakage, label errors, and mismatched splits.

## 5. Hyperparameter search

- **Grid search**: systematic, but wastes trials on unimportant dimensions.
- **Random search**: often a strong baseline when only a few dimensions matter.
- **Successive halving**: allocate small budgets broadly, then more to promising candidates.
- **Bayesian optimization**: use a surrogate to select informative next trials, especially for expensive and low-dimensional tuning.

Tune on validation data or inner cross-validation. Record the search space, random seeds, compute budget, and selection metric. Search itself can overfit, so preserve the final test set.

## 6. Thresholds and calibration

For a probability estimate $p(x)$ and threshold $t$, classify positive when $p(x) \ge t$. The threshold controls the precision/recall tradeoff; 0.5 is not universally optimal. Choose it using validation data and the relative costs of false positives and false negatives.

Calibration asks whether events predicted with probability $q$ occur about a fraction $q$ of the time. A calibrated classifier can still rank poorly, and a good ranker can still be poorly calibrated. Use reliability plots, Brier score, or log loss; calibrate using held-out predictions, never the final test labels.

## 7. Metric choice and slices

For rare positive classes, accuracy can be high while the model misses almost every positive. Report metrics suited to the decision:

- Precision: among predicted positives, how many are true?
- Recall: among actual positives, how many are found?
- F1: harmonic mean of precision and recall at one threshold.
- PR-AUC: ranking behavior emphasizing the positive class; its baseline depends on prevalence.
- ROC-AUC: pairwise ranking across thresholds; may look reassuring under extreme imbalance.
- Log loss/Brier score: probability quality, including confidence.

Always inspect confusion counts, relevant operating thresholds, and performance by meaningful slices such as site, language, device, or time. Slice analysis is diagnostic; small slices have wide uncertainty.

## 8. Worked decision example

Suppose a fraud model catches more fraud but also blocks more legitimate purchases. First pick the unit and cost of an error. Then use validation predictions to plot fraud caught against legitimate blocks over thresholds. Select an operating point that satisfies a predeclared block-rate constraint. Estimate the chosen point once on the untouched test set, including confidence intervals and segment checks.

## 9. Exercises

1. You predict hospital readmission and multiple records belong to each patient. Which split should you use and why?
2. A preprocessing step normalizes features using the mean over the entire dataset before five-fold cross-validation. Identify the flaw and fix.
3. A rare-event classifier has 99% accuracy and 2% recall. What additional metrics and counts do you need?
4. A model wins on a random split but loses on the next month of data. What does that tell you about the evaluation design?

## 10. Check your understanding

1. Use patient-grouped splits; otherwise patient-specific patterns can leak across folds.
2. Put preprocessing in a pipeline and fit it independently inside each training fold.
3. Inspect prevalence, confusion matrix, precision, PR-AUC, calibration, and the cost at candidate thresholds.
4. Random-split performance did not estimate temporal deployment well. Diagnose drift and validate with rolling or forward time splits.

## References

- [scikit-learn: Cross-validation](https://scikit-learn.org/stable/modules/cross_validation.html)
- [scikit-learn: Common pitfalls and recommended practices](https://scikit-learn.org/stable/common_pitfalls.html)
- [scikit-learn: Model evaluation](https://scikit-learn.org/stable/modules/model_evaluation.html)
