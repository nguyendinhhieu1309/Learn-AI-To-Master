# Logistic Regression, Perceptron, and Softmax

These linear classifiers share a score $w^Tx+b$ but turn it into a decision in different ways. Understanding that distinction clarifies their losses, outputs, and failure modes.

## Learning objectives

- Distinguish a linear score, a probability, and a predicted class.
- Derive the binary logistic model and binary cross-entropy.
- Explain how the perceptron differs from logistic regression.
- Extend binary classification to mutually exclusive classes with softmax.
- Choose a threshold using validation data and the costs of errors.

## 1. Scores, probabilities, decisions

For binary input $x$, the linear score is $z=w^Tx+b$. This score can be any real value, so it is not automatically a probability. Logistic regression maps it through the sigmoid:

$$
\sigma(z)=\frac{1}{1+e^{-z}},
\qquad
p=P(y=1\mid x)=\sigma(w^Tx+b).
$$

The complementary probability is $P(y=0\mid x)=1-p$. A threshold $t$ converts probability to a decision:

$$
\hat y=\begin{cases}1 & p\ge t,\\0 & p<t.\end{cases}
$$

The common default $t=0.5$ corresponds to equal costs and a calibrated probability, but business, safety, or medical costs may require another threshold. Choose it using validation data and the expected consequences of false positives and false negatives.

## 2. Log-odds are linear

Taking log odds gives

$$
\log\frac{p}{1-p}=w^Tx+b.
$$

Each coefficient shifts log-odds by a fixed amount when one feature changes by one unit and other features stay fixed. The probability change is not constant because the sigmoid is curved.

## 3. Binary cross-entropy and gradient

For $y\in\{0,1\}$, the Bernoulli likelihood is $p^y(1-p)^{1-y}$. Its negative log-likelihood is binary cross-entropy:

$$
\ell(p,y)=-\left[y\log p+(1-y)\log(1-p)\right].
$$

For $n$ examples, average the losses. With logits $z=Xw+b\mathbf{1}$ and probabilities $p=\sigma(z)$, the gradient is especially simple:

$$
\nabla_w J=\frac{1}{n}X^T(p-y),
\qquad
\frac{\partial J}{\partial b}=\frac{1}{n}\mathbf{1}^T(p-y).
$$

The residual is prediction probability minus target. If a positive example is currently assigned too little probability, $p-y<0$, and gradient descent moves parameters toward increasing its score.

### Hand check for one observation

Let $x=2,y=1,w=0,b=0$. Then $z=0,p=0.5$, and loss $-\log(0.5)\approx0.693$. Gradients are $\partial J/\partial w=(p-y)x=-1$ and $\partial J/\partial b=-0.5$. With learning rate $0.1$, one update gives $w=0.1,b=0.05$ and raises the logit for this positive example from 0 to 0.25.

## 4. Why not use the perceptron update?

The perceptron uses a hard sign decision, commonly $\hat y=\operatorname{sign}(w^Tx+b)$ with labels $y\in\{-1,+1\}$. When it misclassifies an example, a classic update is

$$
w\leftarrow w+\eta yx,
\qquad b\leftarrow b+\eta y.
$$

The perceptron focuses on correcting classification errors and does not produce calibrated probabilities. Logistic regression uses a smooth probabilistic objective and updates even examples it classifies correctly when their probabilities are poor. Both have linear decision boundaries unless features are transformed.

## 5. Multiclass softmax regression

For $K$ mutually exclusive classes, each example receives $K$ logits $z_k=w_k^Tx+b_k$. Softmax converts them into a probability distribution:

$$
p_k=\frac{e^{z_k}}{\sum_{j=1}^{K}e^{z_j}},
\qquad \sum_{k=1}^{K}p_k=1.
$$

For a one-hot label $y_k$, categorical cross-entropy is

$$
\ell=-\sum_{k=1}^{K}y_k\log p_k=-\log p_{\text{true class}}.
$$

The logit gradient is $\partial\ell/\partial z_k=p_k-y_k$. Implementations usually combine log-softmax and cross-entropy for numerical stability. Pass raw logits when the framework loss expects logits; applying softmax twice changes the calculation.

Softmax assumes one correct class per example. If several labels may simultaneously be true, use independent sigmoid outputs and binary cross-entropy per label (multi-label classification), not a softmax that forces probabilities to sum to one.

## 6. Class imbalance, thresholds, and calibration

Accuracy can look high when one class dominates. Inspect confusion matrix, precision, recall, PR curve, and the actual number and cost of errors. Class weighting or resampling can help training, but changes to sampling may affect probability calibration. If probabilities drive a decision or risk estimate, evaluate calibration on representative validation data (for example, reliability diagrams or Brier score).

Do not choose a threshold from test labels and then report performance on the same test set. Keep the test set for the final, one-time evaluation.

## 7. Python pattern

```python
from sklearn.linear_model import LogisticRegression
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import StandardScaler

model = make_pipeline(
    StandardScaler(),
    LogisticRegression(class_weight="balanced", max_iter=1000),
)
model.fit(X_train, y_train)
probability = model.predict_proba(X_valid)[:, 1]
```

Choose `class_weight` only when it matches the objective; it is not a universal fix. Tune any decision threshold on `X_valid`, then evaluate the complete chosen procedure once on the test split.

## Practice with answer checks

1. If $z=0$, what is $\sigma(z)$? **Answer:** $0.5$.
2. For true label 1 and predicted probability 0.8, what is binary cross-entropy? **Answer:** $-\log(0.8)\approx0.223$.
3. Why does a binary classifier with a threshold of 0.5 not necessarily maximize recall? **Answer:** it may need a lower threshold; the decision threshold trades false-positive against false-negative costs.
4. Can softmax represent two positive labels at once? **Answer:** no; its class probabilities sum to one and represent mutually exclusive classes.

## Common mistakes

- Calling logits probabilities.
- Reporting a threshold-dependent metric without stating the threshold.
- Using accuracy alone on an imbalanced dataset.
- Applying softmax before a cross-entropy function that expects raw logits.
- Assuming a high classification score implies well-calibrated probabilities.

## Further reading

- [scikit-learn: LogisticRegression](https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LogisticRegression.html)
