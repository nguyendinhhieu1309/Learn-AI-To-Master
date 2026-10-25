# Ensembles and Boosting

An ensemble combines multiple predictors. The useful question is not “does more models mean better?” but “what kinds of errors are being combined, and what does that cost?”

## Learning objectives

- Distinguish bagging, random forests, boosting, voting, and stacking.
- Explain why averaging decorrelated high-variance trees can help.
- Derive one gradient-boosting step for squared-error regression.
- Compare models using the same split, metric, and tuning budget.
- Interpret feature importance without treating it as causal evidence.

## Prerequisites

Study [decision trees](12-decision-trees-and-naive-bayes.md), [validation](04-generalization-validation-and-model-selection.md), and [evaluation](02-evaluation-and-error-analysis.md) first.

## 1. The basic ensemble idea

For predictors $f_1,\ldots,f_M$, a regression average is:

$$
\hat f(x)=\frac1M\sum_{m=1}^{M}f_m(x).
$$

If errors are not perfectly correlated, averaging can reduce variance. If every model makes the same error, averaging changes nothing. A complex ensemble can also reduce bias or improve ranking, but its benefit must be measured on data that did not guide its construction.

For binary classification, a soft-voting ensemble averages class probabilities:

$$
\hat p(y=1\mid x)=\sum_{m=1}^{M}\alpha_m \hat p_m(y=1\mid x),
\qquad \alpha_m\geq0,\quad\sum_m\alpha_m=1.
$$

This assumes the member probabilities are meaningful and sufficiently comparable. Averaging hard class labels discards confidence information.

## 2. Bagging and random forests

**Bagging** trains each model on a bootstrap sample: draw $n$ training rows with replacement from the original $n$. The prediction averages the models.

A **random forest** adds random feature subsets at candidate splits. This decreases similarity between trees. The intended effect is to retain useful individual trees while reducing correlation among their errors.

If each tree has error variance $\sigma^2$ and pairwise error correlation $\rho$, the variance of an average of $M$ trees is approximately:

$$
\operatorname{Var}(\bar f)=\sigma^2\left(\rho+\frac{1-\rho}{M}\right).
$$

As $M$ grows, the uncorrelated part shrinks, but the correlated part remains. This explains why “add more trees” eventually has diminishing returns.

### Out-of-bag estimates

Each bootstrap tree leaves out roughly one third of training rows on average. Those rows can provide an out-of-bag estimate for that tree. It is useful for diagnostics, but it is still based on the training sample and should not replace an untouched final test set.

## 3. Boosting builds a sequence

Boosting adds weak learners sequentially. Each new learner aims to improve the current model where it performs poorly.

For differentiable loss $L(y,F(x))$, gradient boosting fits the next learner to the negative gradient of the loss with respect to the current prediction:

$$
r_{im}=-\left.\frac{\partial L(y_i,F(x_i))}{\partial F(x_i)}\right|_{F=F_{m-1}},
\qquad
F_m(x)=F_{m-1}(x)+\eta h_m(x).
$$

Here $h_m$ is a small tree and $\eta$ is the learning rate. For squared error $L=\frac12(y-F)^2$, the negative gradient is the residual $y-F$.

### Worked one-step example

Suppose three ordered examples have targets $y=(1,3,5)$ and a constant initial prediction $F_0=3$. Residuals are $(-2,0,2)$. A depth-one tree separates the first two examples from the third and predicts the mean residual in each leaf:

$$
h(x)=(-1,-1,2).
$$

With learning rate $\eta=0.5$:

$$
F_1=F_0+0.5h=(2.5,2.5,4).
$$

The initial sum of squared errors is $(-2)^2+0^2+2^2=8$. After the update, it is $(-1.5)^2+(0.5)^2+1^2=3.5$. The next tree would be fitted to the remaining residuals. This miniature example shows the update rule, not a complete model-selection procedure.

## 4. Bias, variance, and regularization knobs

Typical controls include:

- **tree depth or leaf count:** limits interaction complexity;
- **learning rate:** shrinks each boosting update;
- **number of boosting stages:** controls how many corrections are added;
- **row or feature subsampling:** adds randomness;
- **minimum samples per leaf:** avoids leaves supported by too little data;
- **early stopping:** uses a validation procedure to stop before overfitting.

These are not independent magic controls. A smaller learning rate often needs more stages. Select them with a validation plan, not by repeatedly peeking at the test set.

## 5. Which model should be tried?

| Situation | Good first comparison |
| --- | --- |
| Small, clean tabular data | linear/logistic model and a shallow tree |
| Nonlinear tabular relationships | random forest and gradient-boosted trees |
| Many sparse text features | linear model before a large ensemble |
| Very small dataset | simple regularized model; carefully validate variance |
| Strict latency or explanation constraints | include a simpler model in the comparison |

These are starting points, not guarantees. Data volume, missingness, feature types, latency, and the evaluation split can change the result.

## 6. Feature importance is not explanation by itself

Impurity-based tree importance can favor features with many possible split points. Correlated features can share or obscure importance. Permutation importance asks how a chosen score changes when one feature is shuffled, but correlated features can substitute for one another and make each look unimportant.

Always state:

- which metric was measured;
- which dataset partition was used;
- whether the importance is global or for a subgroup;
- what correlation or leakage risks remain.

An importance score describes model reliance under a procedure. It does not show that changing the feature would cause an outcome to change.

## Experiment

On one fixed tabular split, compare:

1. a mean or majority baseline;
2. a regularized linear model;
3. a single decision tree;
4. a random forest;
5. gradient-boosted trees.

Keep preprocessing, metric, validation strategy, and tuning budget comparable. Record training and validation scores, prediction latency, model size, and error slices. Use the final test set only after choosing the workflow.

## Practice with answer checks

1. Why can averaging 100 nearly identical trees have little value?
2. What changes between bagging and boosting?
3. In squared-error boosting, what target is a new learner fitted to?
4. If a deeper tree improves training error but worsens validation error, what likely changed?
5. Why can impurity importance overstate a high-cardinality feature?
6. Why does a strong ensemble still need a baseline comparison?

### Answers

1. Their errors are highly correlated, so averaging does not cancel much error.
2. Bagging learners are trained independently on resampled data and averaged. Boosting learners are added sequentially to improve the current fit.
3. The current residual $y_i-F_{m-1}(x_i)$.
4. The model gained complexity and variance; it may be fitting sample-specific patterns.
5. It has more candidate split points and more chances to achieve an apparent impurity reduction.
6. A baseline reveals whether complexity produces meaningful gain for the intended metric and use case.

## Common mistakes

- Calling all tree ensembles “random forests.”
- Tuning the number of models until the test score looks best.
- Assuming a small training loss implies a useful model.
- Treating feature importance as causal evidence.
- Comparing models with different splits or unequal tuning effort.

## References

- [scikit-learn: Ensemble methods](https://scikit-learn.org/stable/modules/ensemble.html) — bagging, forests, boosting, voting, and stacking.
- [scikit-learn: RandomForestClassifier](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestClassifier.html) — forest parameters and out-of-bag options.
