# Machine Learning

## Outcomes

- Convert a real question into a target, feature set, baseline, metric, and deployment decision.
- Derive the objectives behind linear/logistic regression, nearest neighbors, k-means, SVMs, trees, and matrix factorization.
- Select a split and metrics that match how future data will arrive.
- Diagnose underfitting, overfitting, leakage, class imbalance, calibration, and distribution shift.
- Explain what a model's assumptions allow it to learn and where its predictions can fail.

## Prerequisites

Python fundamentals, data processing, and basic linear algebra and probability. Review [math topics](../01-math/README.md) as needed.

## Suggested sequence

### A. Start with problem framing

1. [Supervised learning and useful baselines](01-supervised-learning.md)
2. [Evaluation and error analysis](02-evaluation-and-error-analysis.md)
3. [Generalization, validation, and model selection](04-generalization-validation-and-model-selection.md)

### B. Learn core algorithms in this order

4. [Linear regression and gradient descent](07-linear-regression-and-gradient-descent.md)
5. [Logistic regression, perceptron, and softmax](08-logistic-regression-perceptron-and-softmax.md)
6. [k-NN and k-means](09-knn-and-k-means.md)
7. [Decision trees and Naive Bayes](12-decision-trees-and-naive-bayes.md)
8. [Support vector machines](10-support-vector-machines.md)
9. [PCA, SVD, and dimensionality reduction](11-pca-svd-and-dimensionality-reduction.md)
10. [Recommendation systems](13-recommendation-systems.md)

### C. Broaden and consolidate

11. [Unsupervised learning overview](03-unsupervised-learning.md)
12. [Classical algorithms and model choice](05-classical-ml-algorithms-and-model-choice.md)
13. [Unsupervised learning and anomaly detection](06-unsupervised-learning-and-anomaly-detection.md)
14. [Ensembles and boosting](14-ensembles-and-boosting.md)
15. [Probability calibration, decision thresholds, and uncertainty](15-calibration-and-uncertainty.md)

For each algorithm, reproduce the worked example by hand first, then implement a small version, compare it with a trusted library, and inspect errors on a validation split. Use the math [calculus and optimization lesson](../01-math/02-calculus-and-optimization.md) when a gradient step is unclear and the [likelihood lesson](../01-math/08-likelihood-mle-and-map.md) to understand why many common losses have their form.

## Practice

- Complete the [machine learning problem set with worked solutions](PROBLEM_SET_WITH_SOLUTIONS.md) after the algorithm lessons.
- Predict a simple target with a constant baseline before training a model.
- Compare precision, recall, F1, and a confusion matrix at multiple thresholds.
- Evaluate a regression model with more than one metric and inspect residuals.
- Compare random and time-aware validation on a temporal dataset.
- Fit linear regression both with least squares and gradient descent; verify the gradients numerically on a tiny dataset.
- Compare logistic regression and an RBF SVM on the same scaled dataset, then inspect calibration and threshold effects.
- Run k-means with multiple seeds and explain which cluster properties are stable.
- Create a time-based recommender evaluation and explain what its logged interactions do not reveal.

## Checkpoint project

Build a complete tabular prediction study: define the task, document the dataset, establish a baseline, compare at least two model families using a valid split, analyze errors, and write a short conclusion that includes limitations.

The [machine-learning formula cheatsheet](../01-math/FORMULA_CHEATSHEET.md) is a review aid, not a replacement for deriving the objectives in the lessons.

## Common mistakes to watch for

- Selecting a model based only on training score.
- Tuning repeatedly on the test set.
- Choosing accuracy for a highly imbalanced task without checking class-specific errors.
- Treating feature importance as proof of cause.
