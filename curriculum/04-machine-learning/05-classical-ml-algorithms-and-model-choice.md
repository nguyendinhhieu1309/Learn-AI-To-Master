# Classical ML Algorithms and Model Choice

No algorithm wins on every dataset. This lesson explains common supervised methods, their assumptions, and how to build a fair comparison.

## 1. Linear models

Linear regression predicts a weighted sum. Logistic regression maps a linear score through a sigmoid or softmax to estimate class probabilities. Linear models are fast, interpretable, and strong baselines; they may need feature transforms or interactions for nonlinear structure.

Regularization controls coefficient size. L1 can set some coefficients exactly to zero; L2 shrinks all coefficients. Tune regularization on validation data and scale features when the penalty should treat coefficients comparably.

## 2. Decision trees

A decision tree recursively partitions feature space using splits that improve an impurity criterion, such as Gini impurity or entropy. Trees handle nonlinear interactions and mixed feature types with little scaling, but deep trees can memorize training examples. Control depth, leaf size, and pruning; inspect stability under small data changes.

## 3. Bagging and random forests

Bagging trains models on bootstrap samples and aggregates predictions to reduce variance. Random forests also consider a random subset of features at each split, making trees less correlated. They are robust general-purpose tabular baselines, though very high-cardinality categories and extrapolation beyond training ranges can be problematic.

## 4. Gradient-boosted trees

Boosting adds weak learners sequentially to correct residual errors or optimize a differentiable objective. Gradient-boosted tree libraries often perform very well on structured data. Key settings include number of trees, learning rate, depth/leaves, row/feature subsampling, regularization, and early stopping.

Low learning rates generally require more trees. Tune with a validation split that reflects time and groups. Beware leakage in target-derived features and high-cardinality encodings.

## 5. k-nearest neighbors and naive Bayes

k-NN predicts from nearby examples. It is simple and can model local patterns, but requires meaningful scaled distances and can be slow or memory-heavy at inference. Naive Bayes applies Bayes’ rule with conditional independence assumptions; despite the strong assumption, it is an efficient text baseline and can work well with sparse counts.

## 6. Fair model comparison

Use the same data splits, target definition, preprocessing boundaries, and primary metric. Compare against a trivial baseline (majority class, mean, or previous value) and a simple linear model. Record training and inference cost, calibration, error slices, and uncertainty—not just the top score.

## 7. Choosing an algorithm

- Sparse high-dimensional text: linear model or naive Bayes baseline.
- Medium-sized tabular data: boosted trees and regularized linear model are strong candidates.
- Smooth geometry or image/text inputs: neural networks may be suitable.
- Need transparent rules: shallow trees or constrained linear models, with careful validation.
- Need low-latency CPU inference: benchmark compact models under actual load.

These are starting hypotheses, not laws. Data quality and evaluation design often matter more than the algorithm family.

## 8. Exercises

1. Why can a deep decision tree have excellent training score and poor test score?
2. A k-NN model uses age in years and income in dollars. What must be considered before computing distance?
3. Why is a majority-class baseline useful for an imbalanced classifier?
4. A boosted model wins overall but loses substantially for a rare group. What should the model-selection report include?

## References

- [scikit-learn: Supervised learning](https://scikit-learn.org/stable/supervised_learning.html)
- [XGBoost documentation](https://xgboost.readthedocs.io/en/stable/)
