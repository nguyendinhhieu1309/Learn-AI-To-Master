# Support Vector Machines

A support vector machine (SVM) is a margin-based model. Its geometric view explains the objective, its sensitivity to scaling, and how kernel functions create nonlinear boundaries.

## Learning objectives

- Write a linear decision boundary and measure point-to-boundary distance.
- Explain the maximum-margin objective and support vectors.
- Distinguish hard-margin and soft-margin SVMs.
- Interpret hinge loss, the regularization tradeoff, and kernel similarity.
- Know when an SVM is a sensible choice and how to evaluate it.

## 1. A separating hyperplane

For features $x\in\mathbb{R}^{d}$, a linear boundary is

$$
w^Tx+b=0.
$$

The predicted side is determined by the sign of $w^Tx+b$. The perpendicular distance from a point $x$ to the boundary is

$$
\frac{|w^Tx+b|}{\|w\|_2}.
$$

For labels $y_i\in\{-1,+1\}$, the signed margin is $y_i(w^Tx_i+b)/\|w\|_2$. A positive value means the example is on the correct side; a larger value means it is farther from the boundary.

## 2. Maximum margin

If data are perfectly linearly separable, scale $w,b$ so the nearest points satisfy $y_i(w^Tx_i+b)\geq1$. Maximizing geometric margin is equivalent to minimizing the weight norm:

$$
\min_{w,b}\frac12\|w\|_2^2
\quad\text{subject to}\quad
y_i(w^Tx_i+b)\geq1\quad\forall i.
$$

The factor $1/2$ simplifies derivatives. The **support vectors** are the training points on or within the margin; they determine the fitted boundary. Points far beyond the margin usually do not change the exact hard-margin solution.

## 3. Soft margins and hinge loss

Real datasets overlap and contain noise. Introduce slack variables $\xi_i\geq0$ for margin violations:

$$
\min_{w,b,\xi}\frac12\|w\|_2^2+C\sum_i\xi_i
\quad\text{subject to}\quad
y_i(w^Tx_i+b)\geq1-\xi_i.
$$

An equivalent objective uses hinge loss:

$$
\min_{w,b}\frac12\|w\|_2^2
+C\sum_i\max\left(0,1-y_i(w^Tx_i+b)\right).
$$

$C$ controls the tradeoff. Large $C$ penalizes margin violations strongly and may fit the training set more tightly; small $C$ gives stronger regularization and can allow more violations. Libraries differ in whether the objective is written as a sum or average, so do not compare raw $C$ values across implementations without checking conventions.

### Tiny margin check

For $y=+1$ and score $w^Tx+b=1.4$, hinge loss is $\max(0,1-1.4)=0$. For a negative example $y=-1$ with score $0.2$, the signed margin is $-0.2$ and loss is $1.2$. The second example is misclassified and receives a large penalty.

## 4. Kernels: a nonlinear boundary without explicit features

Suppose a transformation $\phi(x)$ maps inputs to a richer feature space. A linear boundary in that space can be nonlinear in the original input. Kernel methods use inner products

$$
K(x,z)=\phi(x)^T\phi(z)
$$

without explicitly computing every transformed feature. Common kernels include:

- Linear: $K(x,z)=x^Tz$.
- Polynomial: $K(x,z)=(\gamma x^Tz+r)^p$.
- Radial basis function: $K(x,z)=\exp(-\gamma\|x-z\|_2^2)$.

The RBF parameter $\gamma$ controls locality. A very large $\gamma$ creates highly local, irregular boundaries; a very small $\gamma$ behaves more smoothly and may underfit. Tune kernel, $C$, and $\gamma$ on validation data, using a pipeline that scales features inside each fold.

## 5. Multi-class predictions and probabilities

The basic SVM is binary. Multi-class packages often combine one-versus-rest or one-versus-one classifiers. Raw decision scores are not calibrated probabilities. If a downstream system needs probabilities, use a calibration procedure fitted with validation data and evaluate calibration separately from ranking quality.

## 6. When to use an SVM

SVMs can be effective for small-to-medium feature datasets, especially with a well-chosen representation and margin. Kernel SVM training can scale poorly with the number of examples and may be cumbersome to deploy at very large scale. For large datasets, linear models, tree ensembles, or neural networks may be more practical. Compare against a simple baseline with a valid split.

## 7. Practice with answer checks

1. For $w=(3,4)$, $b=0$, and $x=(0,2)$, what is the distance to the boundary? **Answer:** $|8|/5=1.6$.
2. What happens to the soft-margin penalty when $C$ is very large? **Answer:** margin violations become expensive, so the fit prioritizes them more strongly.
3. Does a kernel SVM's decision score automatically equal a calibrated probability? **Answer:** no.
4. Why scale features before an RBF-kernel SVM? **Answer:** Euclidean distances and the kernel values depend on coordinate scales.

## Common mistakes

- Forgetting the $\|w\|$ denominator when interpreting geometric distance.
- Assuming hard-margin separation is possible on noisy real data.
- Tuning the kernel on the test set.
- Comparing $C$ values across different objective normalizations as if they were identical.
- Treating a decision score as probability.

## Further reading

- [scikit-learn: SVC](https://scikit-learn.org/stable/modules/generated/sklearn.svm.SVC.html)
