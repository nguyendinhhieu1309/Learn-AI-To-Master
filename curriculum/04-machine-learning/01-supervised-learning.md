# Supervised Learning and Useful Baselines

## Learning objectives

Frame an input-output task, build a baseline, and understand the basic behavior of common supervised models.

## 1. What is supervised learning?

We are given examples $(x_i,y_i)$, where $x_i$ contains input features and $y_i$ is a known target. The model learns a function $f(x)$ that predicts targets for examples it has not seen. In **regression**, targets are numerical. In **classification**, targets are categories.

Write the task before the code: What will be known at prediction time? What outcome matters? What does a mistake cost? How will a prediction be used?

## 2. Always establish a baseline

A baseline is a simple reference. For regression, predict the training-set mean or median. For classification, predict the most common class. A domain rule may be a stronger baseline. If a complex model cannot beat a sensible baseline on untouched data, investigate the data and task before adding complexity.

## 3. Linear regression

The model forms a weighted sum of features:

$$
\hat y=w^Tx+b.
$$

Training often minimizes mean squared error:

$$
\operatorname{MSE}=\frac{1}{n}\sum_{i=1}^{n}(y_i-\hat y_i)^2.
$$

A coefficient describes the model's change in prediction for a one-unit feature increase **holding other included features fixed**; correlated features make this interpretation unstable, and it is not automatically causal.

Feature scales matter for optimization and regularization. Standardization transforms a feature as

$$
x_{\mathrm{scaled}}=\frac{x-\mu_{\mathrm{train}}}{s_{\mathrm{train}}};
$$

compute $\mu_{\mathrm{train}}$ and $s_{\mathrm{train}}$ on training data only.

## 4. Logistic regression

For binary classification, logistic regression converts a linear score into a probability:

$$
z=w^Tx+b,
\qquad p=P(y=1\mid x)=\sigma(z)=\frac{1}{1+e^{-z}}.
$$

The default class decision often thresholds at 0.5, but this is not always the right threshold. The model is linear in its features even though the output probability is nonlinear.

## 5. Trees and ensembles

A decision tree repeatedly splits feature space using questions such as `age < 30?`. Trees can model nonlinear interactions and need less feature scaling, but deep trees can memorize training examples. Random forests average many randomized trees. Gradient-boosted trees add trees sequentially to correct earlier errors. Ensembles can be strong on tabular data but still need careful validation and error analysis.

## 6. Regularization and model complexity

Too-simple models underfit; overly flexible models can overfit. Regularization, early stopping, tree depth limits, and more data can help. Select settings using validation or cross-validation, not the final test set.

## Worked comparison

Suppose a binary model predicts loan default. Start with a majority-class baseline. Then compare logistic regression and a tree ensemble on the same split. Keep the threshold fixed or justify threshold selection using validation data. Report confusion counts and class-specific precision/recall, not just accuracy.

Here is a compact baseline using a dataset bundled with scikit-learn. The held-out portion is not used to fit the scaler or classifier because both live inside the pipeline.

```python
from sklearn.datasets import load_iris
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import classification_report
from sklearn.model_selection import train_test_split
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import StandardScaler

X, y = load_iris(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.25, random_state=7, stratify=y
)
model = make_pipeline(StandardScaler(), LogisticRegression(max_iter=1000))
model.fit(X_train, y_train)
predicted = model.predict(X_test)
print(classification_report(y_test, predicted))
```

The fixed random seed makes this split repeatable within the same toolchain; it does not show how performance varies across possible samples. The Iris dataset is a teaching example, not evidence that the same score will hold on a real deployment task.

## Practice

Choose a dataset. Write down the prediction-time information and a simple baseline before training. Compare a linear model with a tree-based model. Explain one benefit and one limitation of each using held-out errors.

## Common mistakes

- Calling a model accurate without comparing against a baseline.
- Reading coefficients as causal effects.
- Scaling with statistics computed from the full dataset.
- Choosing the best model based on test results after many experiments.
