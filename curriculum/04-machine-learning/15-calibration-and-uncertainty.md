# Probability Calibration, Decision Thresholds, and Uncertainty

A classifier can rank examples well while reporting probabilities that are systematically too high or too low. A decision system also needs a threshold that reflects the costs of its errors. This lesson separates those questions.

## Learning objectives

- Distinguish discrimination, calibration, and classification decisions.
- Compute Brier score and log loss for simple predictions.
- Choose a threshold from explicit error costs.
- Fit a calibrator without leaking validation or test information.
- Explain the assumptions and limitations of a basic conformal interval.

## Prerequisites

Understand logistic regression, confusion matrices, validation, and probability. Review [evaluation and error analysis](02-evaluation-and-error-analysis.md) and [model selection](04-generalization-validation-and-model-selection.md).

## 1. Three different properties

### Discrimination

Can the model rank a randomly chosen positive example above a randomly chosen negative example? AUC summarizes ranking across thresholds. AUC does not say that a score of 0.8 means an 80% event rate.

### Calibration

Among examples assigned probability near $p$, does the event occur about a fraction $p$ of the time?

$$
P(Y=1\mid\hat p\approx p)\approx p.
$$

Calibration is a population statement. A finite dataset can only estimate it with uncertainty.

### Decision quality

Given a probability, threshold, and costs, does the action produce acceptable outcomes? The threshold depends on the problem, not just the model.

## 2. Proper scoring rules

For binary label $y\in\{0,1\}$ and probability $p$:

$$
\operatorname{Brier}(y,p)=(p-y)^2,
\qquad
\operatorname{LogLoss}(y,p)=-y\log p-(1-y)\log(1-p).
$$

Both reward honest probabilities in expectation under the data distribution. Log loss penalizes confident wrong predictions very strongly; numerical implementations clip probabilities away from exactly 0 and 1.

### Worked example

For outcomes $(1,0,1,0)$ and predictions $(0.8,0.6,0.4,0.2)$:

$$
\operatorname{Brier}=\frac{(0.8-1)^2+(0.6-0)^2+(0.4-1)^2+(0.2-0)^2}{4}
=0.25.
$$

The mean log loss is:

$$
\frac{-\log(0.8)-\log(0.4)-\log(0.4)-\log(0.8)}4
\approx0.569.
$$

The second and third examples are confidently wrong and therefore contribute most of the log loss.

## 3. Reliability diagrams and expected calibration error

To inspect calibration:

1. Divide predictions into probability bins.
2. For each bin, plot average predicted probability against observed positive frequency.
3. Show the number of examples in each bin; a bin with five examples is noisy.

Expected calibration error (ECE) is a weighted average of the absolute gap between bin accuracy and bin confidence. ECE depends on binning choices and is not a substitute for a proper scoring rule. Report the binning method and use a reliability plot alongside Brier score or log loss.

## 4. Calibration methods

- **Platt scaling:** fit a logistic mapping from an out-of-sample model score to observed labels. It is smooth and compact.
- **Isotonic regression:** fit a non-decreasing mapping without assuming a sigmoid shape. It can be flexible but needs more calibration data to avoid overfitting.
- **Temperature scaling:** adjust the scale of logits while preserving the predicted class ranking in common multiclass settings.

Calibration data must be separate from the data used to fit the base model, or generated with a cross-validation procedure that keeps predictions out of fold. Do not fit a calibrator to in-sample predictions and then claim it estimates future calibration.

## 5. Derive a cost-sensitive threshold

Suppose action 1 means “intervene.” Let $C_{FP}$ be the cost of intervening on a negative case and $C_{FN}$ the cost of not intervening on a positive case. Under calibrated probability $p$:

- expected cost of action 1: $(1-p)C_{FP}$;
- expected cost of action 0: $pC_{FN}$.

Choose action 1 when:

$$
(1-p)C_{FP}<pC_{FN}
\quad\Longleftrightarrow\quad
p>\frac{C_{FP}}{C_{FP}+C_{FN}}.
$$

If a false negative costs 4 units and a false positive costs 1, the threshold is $1/(1+4)=0.2$. This result assumes the probabilities and costs apply to the deployment population and that the action has no additional constraints.

## 6. Prior shift and monitoring

If class prevalence changes while the class-conditional score distributions remain stable, calibration can drift even when ranking is similar. In production, monitor:

- calibration by time window and important subgroup;
- Brier or log loss with confidence intervals;
- prevalence and missingness changes;
- action volume at the selected threshold;
- delayed-label maturity.

Recalibration must use labels that have matured and a procedure that respects time. Do not infer performance from unlabeled traffic alone.

## 7. Prediction intervals with split conformal regression

Split conformal prediction gives a simple way to form intervals under exchangeability:

1. Fit a regression model on a training set.
2. On a separate calibration set, compute residual scores $s_i=|y_i-\hat y_i|$.
3. For target miscoverage $\alpha$, choose the corrected empirical quantile with rank $\lceil(n_{\mathrm{cal}}+1)(1-\alpha)\rceil$.
4. Return $[\hat y(x)-q,\hat y(x)+q]$.

Under exchangeability and the stated split procedure, the guarantee is marginal coverage over future examples, not exact conditional coverage for every subgroup or every $x$. Time dependence, distribution shift, and data leakage can invalidate the guarantee. The interval can be wide; coverage alone does not mean it is useful.

## Practice

1. Two models have equal AUC, but one has worse log loss. What might differ?
2. A model has predicted probability 0.8 on 50 similar validation examples and 30 positives. Is that bin calibrated?
3. Compute the threshold when false positives cost 3 and false negatives cost 2.
4. Why is calibration on the training predictions optimistic?
5. What does a 90% conformal interval guarantee, and what does it not guarantee?

## Answer checks

1. Their ranking may be similar while one assigns overconfident or underconfident probabilities.
2. The observed rate is $30/50=0.6$, below the predicted 0.8; this bin suggests overconfidence, subject to sampling uncertainty.
3. $C_{FP}/(C_{FP}+C_{FN})=3/5=0.6$.
4. The base model has already fit those labels, so its scores do not represent genuinely unseen predictions.
5. Under exchangeability and correct separation, it aims for approximately 90% marginal coverage of future labels. It does not promise 90% coverage within every subgroup or under arbitrary distribution shift.

## Common mistakes

- Calling an AUC score “confidence.”
- Changing a threshold without stating costs or operational constraints.
- Using the same data to train, calibrate, tune, and report final performance.
- Treating ECE as independent of binning choices.
- Claiming conformal guarantees after time leakage or population shift.

## References

- [scikit-learn: Probability calibration](https://scikit-learn.org/stable/modules/calibration.html) — calibration curves and calibration procedures.
- [scikit-learn: Model evaluation](https://scikit-learn.org/stable/modules/model_evaluation.html) — probability losses and classification metrics.
