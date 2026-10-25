# Vectors and Matrices

## Learning objectives

You will learn how vectors and matrices represent data, how dot products produce useful scores, and why dimensions matter in model code.

## 1. Vectors represent lists with meaning

A vector is an ordered list of numbers. A person's features could be `x = [age, height, weight]`. The order matters: `[age, height]` does not mean the same thing as `[height, age]`.

The **shape** tells us how many values there are. This course writes a column vector as

$$
x = \begin{bmatrix}2\\3\\4\end{bmatrix}, \qquad x \in \mathbb{R}^{3}.
$$

The real numbers $\mathbb{R}$ tell us the values are real; the exponent 3 tells us there are three coordinates.

## 2. Dot product: weighted score and similarity

For equal-length vectors, the dot product multiplies corresponding entries and adds them:

$$
w^T x = \sum_{j=1}^{d} w_j x_j.
$$

If $w=[2,-1]$ and $x=[3,4]$, then $w^Tx=2(3)-1(4)=2$. A linear model adds a bias: $\hat y=w^Tx+b$.

The dot product is also related to angle:

$$
u^Tv=\|u\|_2\|v\|_2\cos\theta.
$$

After vectors are normalized to length 1, their dot product (cosine similarity) is 1 for identical direction, 0 for perpendicular directions, and $-1$ for opposite directions.

## 3. Norms measure size

The Euclidean, or $L_2$, norm is

$$
\|x\|_2=\sqrt{\sum_{j=1}^{d}x_j^2}.
$$

For $x=[3,4]$, it is 5. The $L_1$ norm is $\|x\|_1=\sum_j|x_j|$, which is 7 for $[3,4]$. Norms can express distance or constrain model weights. $L_1$ regularization often encourages some weights to become exactly zero; $L_2$ regularization tends to shrink weights smoothly.

## 4. Matrices transform and collect data

A matrix is a rectangular grid. If `X` has `n` rows (examples) and `d` columns (features), then `X` has shape `(n, d)`. Multiplying `Xw`, where `w` has shape `(d,)`, produces `n` scores—one per example.

Matrix multiplication is legal only when inner dimensions match: `(n, d) @ (d, k) -> (n, k)`. A common bug is to multiply incompatible shapes or to transpose an array unnecessarily. Print or inspect shapes before guessing.

A matrix can represent a linear transformation: it rotates, stretches, reflects, or projects vectors. A rotation changes direction while preserving length; a projection keeps only components along selected directions.

## 5. Eigenvectors and PCA intuition

An eigenvector of a square matrix is a direction that the matrix scales without changing its direction:

$$
Av=\lambda v.
$$

Principal component analysis (PCA) finds directions that capture large variance in data. It can compress data, but components are new combinations of features and may be harder to interpret. PCA must be fit on training data only when used before model evaluation.

## Worked example: one batch of predictions

Suppose 3 examples each have 2 features:

\[
X = \begin{bmatrix}1&2\\0&1\\3&1\end{bmatrix}, \quad w=\begin{bmatrix}2\\-1\end{bmatrix}, \quad b=0.5.
\]

Then the scores are $[1(2)+2(-1),\;0(2)+1(-1),\;3(2)+1(-1)]+0.5=[0.5,-0.5,5.5]$. The shapes are $(3,2)\times(2,)\to(3,)$.

## Check your understanding

1. What is the dot product of $[1,2]$ and $[3,4]$? **Answer:** 11.
2. What is the shape of 12 examples with 5 features? **Answer:** `(12, 5)`.
3. If $A$ has shape $(8,4)$ and $B$ has shape $(4,2)$, what is $AB$'s shape? **Answer:** $(8,2)$.

## Common mistakes

- Forgetting that feature order must be identical in training and prediction.
- Confusing element-wise multiplication with matrix multiplication.
- Comparing vector magnitudes when cosine direction was intended, or vice versa.
- Using a dimensionality reduction transform fitted on all data before a split.
