# PCA, SVD, and Dimensionality Reduction

Principal component analysis (PCA) finds orthogonal directions that preserve as much variance as possible under a linear projection. It is useful for compression, visualization, denoising, and as a preprocessing step—but it does not discover labels or guarantee a better model.

## Learning objectives

- Derive the first principal component as a maximum-variance direction.
- Connect covariance eigenvectors to singular value decomposition.
- Calculate explained variance and project/reconstruct a point.
- Fit and use PCA without leaking validation or test information.
- Explain limitations and compare PCA with supervised alternatives.

## 1. Center the training data

Let $X\in\mathbb{R}^{n\times d}$ contain $n$ observations in rows and $d$ features in columns. Compute a mean for each feature using training examples and center them:

$$
X_c=X-\mathbf{1}\mu^T,
\qquad
\mu_j=\frac1n\sum_{i=1}^{n}X_{ij}.
$$

PCA is based on variance around the mean. If the feature means are not removed, the first component may mostly point toward the data offset. The covariance matrix is

$$
S=\frac{1}{n-1}X_c^TX_c\in\mathbb{R}^{d\times d}.
$$

## 2. Derive the first component

Project centered row-vector $x_i$ onto a unit direction $u$. Its scalar coordinate is $x_i^Tu$. The total sample variance of the projected coordinates is

$$
\frac{1}{n-1}\sum_i(x_i^Tu)^2=u^TSu.
$$

The first principal direction solves

$$
u_1=\arg\max_{\|u\|_2=1}u^TSu.
$$

Using a Lagrange multiplier gives $Su_1=\lambda_1u_1$: the direction is an eigenvector of the covariance matrix with the largest eigenvalue. Later components maximize the remaining variance while staying orthogonal to earlier components.

## 3. Worked two-feature example

Take four already centered points

$$
X_c=\begin{bmatrix}1&0\\-1&0\\0&2\\0&-2\end{bmatrix}.
$$

The sample covariance and its eigenvalues are

$$
S=\frac13X_c^TX_c=
\begin{bmatrix}2/3&0\\0&8/3\end{bmatrix},
\qquad \lambda_1=8/3,\;\lambda_2=2/3.
$$

The first component points along the second feature. It preserves $8/10=80\%$ of total variance. A point $(3,4)^T$ projects to coordinate 4 along $u_1=(0,1)^T$ and reconstructs as $(0,4)^T$. The lost horizontal component is why projection is lossy.

## 4. Explained variance and choosing components

If eigenvalues are sorted largest first, the variance retained by the first $k$ components is

$$
\operatorname{explained\ variance\ ratio}(k)
=\frac{\sum_{j=1}^{k}\lambda_j}{\sum_{j=1}^{d}\lambda_j}.
$$

Plot cumulative explained variance and validation performance for the downstream task. A threshold such as 95% is a compression heuristic, not a universal rule for preserving predictive information: a low-variance direction can still contain useful label signal.

## 5. SVD gives a stable computational route

For centered data, compute a thin singular value decomposition:

$$
X_c=U\Sigma V^T.
$$

Columns of $V$ are feature-space principal directions. Singular values $\sigma_j$ relate to covariance eigenvalues by

$$
\lambda_j=\frac{\sigma_j^2}{n-1}.
$$

Keeping the first $k$ columns $V_k$ transforms a row-vector example as $z=x_c^TV_k$ (equivalently, for row-major batches, $Z=X_cV_k$). Reconstruction is $\hat X_c=ZV_k^T$. SVD works for rectangular matrices and is often more numerically stable than forming $X_c^TX_c$ first.

## 6. Scale or do not scale?

PCA on raw units gives larger-variance units more influence. Standardizing each feature to unit variance changes the question to one about correlations rather than raw covariance. Use standardization when differences of units are arbitrary and comparable scale is desired. Do not blindly standardize when absolute magnitude carries meaningful signal. Decide based on the feature definitions, then validate the complete pipeline.

## 7. Avoid leakage

The feature mean, scaling statistics, and principal components are all learned from data. Fit the scaler and PCA **only on each training fold**, then transform validation examples with those fitted objects. In scikit-learn, put `StandardScaler` and `PCA` inside a `Pipeline` with the estimator so cross-validation refits them correctly.

```python
from sklearn.decomposition import PCA
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression

model = make_pipeline(
    StandardScaler(),
    PCA(n_components=0.95),
    LogisticRegression(max_iter=1000),
)
model.fit(X_train, y_train)
```

The `0.95` setting retains enough components to explain 95% of training variance. It does not promise 95% classification accuracy.

## 8. Limitations and alternatives

- PCA is linear and preserves directions of high variance, not necessarily class separation or task-relevant structure.
- Components can be hard to interpret because each combines original features.
- Outliers can strongly affect means and covariance.
- PCA can reduce noise and compute, but can also remove useful signals.
- LDA is supervised and seeks directions that separate labeled classes; it answers a different question.
- Random projections, autoencoders, and nonlinear manifold methods make different assumptions and need task-specific evaluation.

## Practice with answer checks

1. What does eigenvalue $\lambda_j$ represent in covariance PCA? **Answer:** variance along its unit eigenvector.
2. If eigenvalues are $9,1$, what fraction of variance does PC1 retain? **Answer:** $9/(9+1)=90\%$.
3. Does retaining 95% of feature variance guarantee preserving 95% of predictive accuracy? **Answer:** no.
4. Why fit PCA inside each training fold? **Answer:** validation/test data must not influence the learned directions or scaling.

## Further reading

- [scikit-learn: PCA](https://scikit-learn.org/stable/modules/generated/sklearn.decomposition.PCA.html)
