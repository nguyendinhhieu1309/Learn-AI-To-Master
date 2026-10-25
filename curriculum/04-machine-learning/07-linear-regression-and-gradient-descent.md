# Linear Regression and Gradient Descent

Linear regression is the first complete example of an ML model: it has a representation, a loss, an optimization method, assumptions, diagnostics, and a path from hand calculation to a library implementation.

## Learning objectives

- Write a scalar and vectorized linear model and check their shapes.
- Derive squared-error loss and its gradient.
- Solve least squares with a numerical linear algebra routine.
- Explain when gradient descent is useful and how its learning rate behaves.
- Diagnose residual patterns and use regularization responsibly.

## 1. The prediction problem

Suppose we observe $n$ examples. Each row $x_i\in\mathbb{R}^{d}$ contains $d$ features, and $y_i\in\mathbb{R}$ is the target. A linear model predicts

$$
\hat y_i=w^Tx_i+b,
$$

where $w\in\mathbb{R}^{d}$ contains one coefficient per feature and $b$ is the intercept. The coefficient $w_j$ is the model's change in prediction per one-unit change in feature $j$, **holding other features fixed**. This is a description of the fitted model, not a causal claim.

For row-major batches, put the examples in $X\in\mathbb{R}^{n\times d}$:

$$
\hat y=Xw+b\mathbf{1},
\qquad X:(n,d),\;w:(d,),\;\hat y:(n,).
$$

If the intercept is folded into the matrix, append a column of ones to $X$ and append $b$ to $w$. Choose one convention and use it consistently.

## 2. Loss: how wrong are the predictions?

Let residual $r_i=\hat y_i-y_i$. The mean squared error is

$$
\operatorname{MSE}=\frac{1}{n}\sum_{i=1}^{n}r_i^2.
$$

For convenient derivatives, many derivations minimize half the mean squared error:

$$
J(w,b)=\frac{1}{2n}\sum_{i=1}^{n}(w^Tx_i+b-y_i)^2
=\frac{1}{2n}\|Xw+b\mathbf{1}-y\|_2^2.
$$

The factor $1/2$ does not change which parameters minimize the loss. Squaring makes large residuals count more heavily and gives a smooth objective, but it is sensitive to outliers. Compare residual plots and consider MAE or Huber loss when the data support a robust objective.

## 3. Derive the gradient

For a single example, $\ell=\tfrac12(wx+b-y)^2$. The inner prediction has derivatives $\partial(wx+b-y)/\partial w=x$ and $\partial(wx+b-y)/\partial b=1$. By the chain rule,

$$
\frac{\partial\ell}{\partial w}=(wx+b-y)x,
\qquad
\frac{\partial\ell}{\partial b}=wx+b-y.
$$

Averaging over the dataset gives

$$
\nabla_wJ=\frac{1}{n}X^T(Xw+b\mathbf{1}-y),
\qquad
\frac{\partial J}{\partial b}=\frac{1}{n}\mathbf{1}^T(Xw+b\mathbf{1}-y).
$$

The weight gradient has $d$ entries because it needs one direction for every feature. Its matrix dimensions confirm that:

$$
(d\times n)(n\times1)=(d\times1).
$$

### Worked example: compute one update

Use one example $x=2$, $y=5$, and initialize $w=1,b=0$. The prediction is $2$, residual $\hat y-y=-3$, and half-squared loss is $4.5$. The gradient is $(-6,-3)$. With learning rate $\eta=0.1$:

$$
w_{\mathrm{new}}=1-0.1(-6)=1.6,
\qquad b_{\mathrm{new}}=0-0.1(-3)=0.3.
$$

The updated prediction is $3.5$, so its half-squared loss is $1.125$. This step moved in a direction that reduced the loss.

## 4. Gradient descent

Gradient descent repeats the update

$$
w\leftarrow w-\eta\nabla_wJ,
\qquad b\leftarrow b-\eta\frac{\partial J}{\partial b}.
$$

The learning rate $\eta$ controls step size. A rate that is too large may increase loss or oscillate; a rate that is too small may make training painfully slow. Plot the objective against updates and tune on validation behavior, not on a final test set.

- **Batch gradient descent:** calculate each update from all training examples.
- **Stochastic gradient descent:** use one example per update; updates are noisy and frequent.
- **Mini-batch gradient descent:** average gradients over a small batch; this is a common hardware-friendly compromise.
- **Momentum/Adam:** use information from recent gradients to smooth or rescale updates. They do not remove the need to check data, shapes, or learning rate.

Standardizing features using statistics fit on the training set often improves optimization. Fit the scaler inside the training pipeline, then apply the same stored transform to validation, test, and production data.

## 5. The closed-form least-squares solution

Add an intercept column to make $\bar X=[\mathbf{1},X]$ and combine coefficients into $\theta=[b,w^T]^T$. The objective is $\|\bar X\theta-y\|_2^2$. Differentiating and setting the gradient to zero yields the normal equations:

$$
\bar X^T\bar X\theta=\bar X^Ty.
$$

If the matrix has full column rank, this can be written $\hat\theta=(\bar X^T\bar X)^{-1}\bar X^Ty$. In code, prefer `numpy.linalg.lstsq` or a QR/SVD-based solver over explicitly calculating the inverse. If columns are dependent, the solution may not be unique; a least-squares solver returns a stable solution under its documented convention.

### Worked example: fit three points

Given $(x,y)=\{(0,1),(1,3),(2,5)\}$, the points lie exactly on $y=2x+1$. Thus $w=2,b=1$ and each residual is zero. If the observed labels are instead $1,2.8,5.2$, least squares balances the residuals; the fitted line will generally not pass through every point. A model should be judged on held-out data and residual structure, not only how closely it follows training points.

## 6. Feature engineering and model limits

“Linear” means linear in the learned coefficients. It can model curves after transforming the input, for example

$$
\phi(x)=[x, x^2, \sin x],
\qquad \hat y=w^T\phi(x)+b.
$$

The model is still linear in $w$, even though its relationship with the original $x$ is curved. These features should be motivated and checked for overfitting. Correlated or redundant features can make individual coefficients unstable even when predictions are similar.

## 7. Regularization

Ridge regression adds an $L_2$ penalty, commonly excluding the intercept:

$$
J_{\mathrm{ridge}}(w,b)=\operatorname{MSE}(w,b)+\lambda\|w\|_2^2.
$$

Larger $\lambda$ shrinks coefficients more. Lasso uses $\lambda\|w\|_1$ and can set some coefficients exactly to zero. Regularization may improve generalization when the unregularized model is too sensitive, but tune $\lambda$ on validation data and scale features because penalties depend on feature units.

## 8. Residual diagnostics and assumptions

Plot residuals $y-\hat y$ against predictions, each feature, time, and relevant groups. Curvature may indicate a missing nonlinear term; a widening residual spread may indicate nonconstant variance; grouped patterns may indicate missing structure or an invalid split. For inference about coefficients, additional assumptions are needed; for prediction, held-out performance and data representativeness are central.

## 9. Minimal Python implementation

```python
import numpy as np

X = np.array([[0.0], [1.0], [2.0]])  # shape (examples, features)
y = np.array([1.0, 3.0, 5.0])       # shape (examples,)
X_with_bias = np.c_[np.ones(len(X)), X]

theta, residuals, rank, singular_values = np.linalg.lstsq(
    X_with_bias, y, rcond=None
)
b, w = theta
print(f"bias={b:.3f}, weight={w:.3f}")
```

For a production workflow, put the preprocessing and estimator into one `scikit-learn` pipeline and evaluate it using a split that matches the way future examples will arrive.

## Practice with answer checks

1. If $\hat y=wx+b$, which parameter changes all predictions by the same amount? **Answer:** the bias $b$.
2. For $x=3$, $y=4$, $w=1$, $b=0$, what are the prediction and single-example half-squared loss? **Answer:** $3$ and $0.5$.
3. Why can the direct normal-equation inverse be unreliable? **Answer:** rank deficiency or poor conditioning can make the inverse nonexistent or numerically unstable; solve the least-squares problem with QR/SVD.
4. A training residual plot shows a U-shape. What is one plausible follow-up? **Answer:** examine a justified nonlinear feature or a more suitable model, then evaluate it on valid held-out data.

## Common mistakes

- Forgetting the intercept or using different preprocessing at train and inference time.
- Confusing a small training loss with good future predictions.
- Using the explicit inverse of $X^TX$ in production code.
- Treating a coefficient as a causal effect.
- Selecting features, regularization, or metrics using the test set.

## Further reading

- [scikit-learn: LinearRegression](https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LinearRegression.html)
- [NumPy: least-squares solution](https://numpy.org/doc/stable/reference/generated/numpy.linalg.lstsq.html)
