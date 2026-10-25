# Linear Algebra for Machine Learning

This lesson connects vectors and matrices to model parameters, embeddings, projections, and numerical behavior.

## 1. Vectors, norms, and distance

A vector is an ordered list of features. Its L1 and L2 norms measure magnitude:

$$
\|x\|_1=\sum_i |x_i|,\qquad
\|x\|_2=\sqrt{\sum_i x_i^2}
$$

L1 penalties encourage sparse coefficients; L2 penalties shrink coefficients smoothly. Euclidean distance is $\|x-y\|_2$. Feature scaling matters: if one coordinate is measured in thousands and another in fractions, the large-unit feature can dominate distances and regularization.

## 2. Matrices as transformations

A matrix maps an input vector into a new space. In a linear layer, $y=Wx+b$; each row of $W$ defines one output unit. A neural-network layer applies a nonlinearity after this transformation. Track dimensions explicitly: if $x\in\mathbb{R}^{d}$ and there are $h$ hidden units, then $W\in\mathbb{R}^{h\times d}$ and $Wx\in\mathbb{R}^h$.

For batches, choose a convention and keep it consistent. With examples as rows, $X\in\mathbb{R}^{b\times d}$ and $W\in\mathbb{R}^{d\times h}$, so $XW\in\mathbb{R}^{b\times h}$. Shape bugs often reveal swapped batch/channel axes, incorrect broadcasting, or a transposed weight matrix.

## 3. Rank, bases, and projections

The rank of a matrix is the dimension of the space spanned by its columns. Linearly dependent features add no new linear direction. A basis is a set of independent vectors that can represent every vector in a space.

For a unit vector $u$, the scalar projection of $x$ onto $u$ is $u^\top x$, and the projected vector is $(u^\top x)u$. Projection onto a subspace with orthonormal basis matrix $U$ is:

$$
\operatorname{proj}_{U}(x)=UU^\top x
$$

Principal component analysis finds directions of high variance and projects onto a lower-dimensional subspace. Standardize features first when their units or scales differ; do not standardize blindly when absolute scale is meaningful.

## 4. Eigenvalues and singular value decomposition

An eigenvector $v$ of square matrix $A$ retains its direction under the transformation: $Av=\lambda v$. Eigenvalues describe stretching or shrinking along eigen-directions. For covariance matrices, eigenvectors give principal directions and eigenvalues give variance along them.

Singular value decomposition works for any real matrix:

$$
A=U\Sigma V^\top
$$

The diagonal singular values in $\Sigma$ are nonnegative and sorted by convention. Truncating small singular values gives a low-rank approximation, useful for compression and denoising. A large condition number (ratio of largest to smallest nonzero singular value) can make a problem numerically sensitive.

## 5. Tensors and broadcasting

A tensor is an array with named or positional axes. Write down semantic dimensions such as batch, height, width, channel, time, and feature. Broadcasting aligns dimensions from the right under array-library rules; it can make concise code, but a missing singleton axis can silently produce the wrong shape.

Example: subtracting a per-feature mean of shape `(features,)` from a batch matrix `(batch, features)` is intended. Subtracting a per-example mean requires shape `(batch, 1)`. Assert shapes at data boundaries and inspect a tiny example by hand.

## Worked example: covariance, principal components, and projection

Consider four two-dimensional centered observations:

$$
X=\begin{bmatrix}
1&0\\-1&0\\0&2\\0&-2
\end{bmatrix}.
$$

Using sample covariance with denominator $n-1=3$ gives

$$
S=\frac{1}{3}X^TX
=\begin{bmatrix}2/3&0\\0&8/3\end{bmatrix}.
$$

The coordinate axes are eigenvectors. Their eigenvalues $2/3$ and $8/3$ are the sample variances along those directions. The first principal component is the vertical axis because it has the larger eigenvalue. Keeping only that direction preserves

$$
\frac{8/3}{8/3+2/3}=0.8
$$

of the total variance. For a new point $x=(3,4)^T$, its coordinate along the first component $u=(0,1)^T$ is $u^Tx=4$ and its one-dimensional reconstruction is $uu^Tx=(0,4)^T$. The discarded horizontal value represents the information removed by the projection.

In real data, first subtract the training-set mean. Scaling each feature to unit variance changes the geometry and therefore the principal directions, so standardize only when that is appropriate for the meaning of the features.

## Least squares as projection

For the linear system $Xw\approx y$, exact equality may be impossible when observations are noisy or there are more observations than parameters. Least squares chooses $w$ whose prediction $Xw$ is closest to $y$ in Euclidean distance:

$$
\hat w=\arg\min_w\|Xw-y\|_2^2.
$$

At the optimum, the residual $r=y-X\hat w$ is orthogonal to every column of $X$, which yields the normal equations:

$$
X^TX\hat w=X^Ty.
$$

If $X^TX$ is invertible, $\hat w=(X^TX)^{-1}X^Ty$. When columns are dependent or features outnumber examples, an inverse may not exist; use a least-squares solver or pseudoinverse rather than explicitly computing a matrix inverse. Regularized least squares (ridge) adds a penalty to stabilize poorly determined directions:

$$
\hat w_{\mathrm{ridge}}=\arg\min_w\left(\|Xw-y\|_2^2+\lambda\|w\|_2^2\right).
$$

## Singular value decomposition and PCA

For centered data matrix $X\in\mathbb{R}^{n\times d}$, a thin singular value decomposition is

$$
X=U\Sigma V^T.
$$

Columns of $V$ are orthonormal directions in feature space; the singular values in $\Sigma$ describe how much centered data varies along each direction. The sample covariance is

$$
S=\frac{1}{n-1}X^TX
=V\frac{\Sigma^T\Sigma}{n-1}V^T.
$$

So PCA can be computed from covariance eigenvectors or from the right singular vectors of centered data. Direct SVD is often numerically preferable to forming $X^TX$, which squares the condition number. Keeping the top $k$ right singular vectors projects data to the leading $k$-dimensional subspace; reconstructing from them is the best rank-$k$ approximation under squared error.

## 6. Exercises

1. $X$ has shape $(64, 128)$ and a linear layer has 32 outputs. What shape should its weight matrix and output have using row-major batches?
2. Why should features be scaled before using k-nearest neighbors or an L2-regularized linear model?
3. What does a small singular value suggest about a direction in the data?
4. A projection matrix is built from non-orthonormal columns. Why is $UU^\top$ not generally the correct projector?

5. In the worked PCA example, what fraction of variance is discarded by keeping only the first component? **Answer:** $1-0.8=0.2$, or 20%.
6. If $X$ has rank 3, how many singular values are nonzero in exact arithmetic? **Answer:** 3.

## References

- [MIT OpenCourseWare: Linear Algebra](https://ocw.mit.edu/courses/18-06sc-linear-algebra-fall-2011/)
- [NumPy linear algebra](https://numpy.org/doc/stable/reference/routines.linalg.html)
