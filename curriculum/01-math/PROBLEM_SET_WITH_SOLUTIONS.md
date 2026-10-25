# Mathematics for AI: Problem Set with Worked Solutions

This workbook turns formulas from the math lessons into calculations you can check by hand. Try every problem before reading its solution. Keep units, shapes, and assumptions beside the numbers.

## How to use this workbook

For each problem:

1. Write the known quantities and the unknown.
2. State the formula and its assumptions.
3. Calculate by hand.
4. Check the answer with a short program only after the derivation is clear.

## Problems

### 1. Dot products and similarity

Let $a=(1,2)$ and $b=(2,-1)$.

1. Compute $a^\top b$.
2. Compute both Euclidean norms.
3. What does the result say about the angle between the vectors?

### 2. Matrix shapes

Let $X\in\mathbb{R}^{3\times 2}$ contain three examples and two features, and let $w\in\mathbb{R}^{2}$ and $b\in\mathbb{R}$.

1. What is the shape of $Xw$?
2. What is the shape of $\hat y=Xw+b$?
3. Explain why adding a scalar bias to every row is valid.

### 3. One gradient-descent update

Minimize $f(w)=\frac{1}{2}(w-3)^2$. Start at $w_0=0$ with learning rate $\eta=0.2$.

1. Find $f'(w)$.
2. Compute $w_1=w_0-\eta f'(w_0)$.
3. Compare $f(w_0)$ and $f(w_1)$.

### 4. A linear-regression gradient by hand

Use the objective

$$
J(w,b)=\frac{1}{2m}\sum_{i=1}^{m}(wx_i+b-y_i)^2
$$

with two examples $(x,y)=(0,1)$ and $(1,3)$. Start from $w=b=0$.

1. Compute the predictions, errors, and loss.
2. Compute $\partial J/\partial w$ and $\partial J/\partial b$.
3. Make one gradient step with $\eta=0.1$.

### 5. L2 regularization

Suppose $J_{\mathrm{reg}}(w)=J(w)+\frac{\lambda}{2}w^2$.

1. Differentiate the penalty.
2. If $J'(w)=-1.5$, $w=2$, and $\lambda=0.1$, what is the regularized gradient?
3. Why is the intercept commonly excluded from this penalty?

### 6. Bayes' rule and a rare event

A condition affects 1% of a population. A test has sensitivity 90% and false-positive rate 5%. A person tests positive.

1. Calculate the probability of a positive test.
2. Calculate the probability the person has the condition given the positive test.
3. Explain why “90% sensitive” does not mean a positive result implies a 90% chance of disease.

### 7. Mean and variance

For the observations $1,3,5$, compute the mean, population variance, and sample variance. State which denominator you use in each variance.

### 8. Standard error and an approximate confidence interval

A sample has $n=16$, mean $\bar{x}=12$, and sample standard deviation $s=4$. Using the large-sample normal multiplier 1.96 as an approximation:

1. Estimate the standard error of the mean.
2. Compute the approximate 95% interval.
3. Name one reason this approximation may be unreliable.

### 9. Bernoulli MLE and MAP

In 10 independent trials, 7 are successes. Let $p$ be the success probability.

1. Find the maximum-likelihood estimate.
2. Use a $\operatorname{Beta}(2,2)$ prior and find the posterior distribution.
3. Find the posterior mode and compare it with the MLE.

### 10. Cross-entropy

For one binary example with true label $y=1$ and predicted probability $p=0.8$, compute the binary cross-entropy. What happens to the loss as $p$ approaches zero?

### 11. PCA in a simple case

Consider centered two-dimensional observations $(1,0)$ and $(-1,0)$. Using population covariance (divide by $n$):

1. Compute the covariance matrix.
2. Find its eigenvalues and a principal eigenvector.
3. What information is lost by projecting onto the first principal component?

### 12. Finite-difference gradient check

For $f(w)=w^2$ at $w=2$, approximate the derivative using the central difference with $h=0.001$:

$$
\frac{f(w+h)-f(w-h)}{2h}.
$$

Compare it with the analytic derivative.

### 13. Convexity from a Hessian

For $f(x,y)=x^2+3y^2$, compute the gradient and Hessian. Is the function convex? Is it strictly convex?

### 14. Entropy and uncertainty

A fair binary variable has probabilities $(0.5,0.5)$. Compute its entropy in bits. Then compute the entropy for probabilities $(0.9,0.1)$ and explain the difference.

## Worked solutions

### Solution 1

$$
a^\top b=1(2)+2(-1)=0,\qquad
\|a\|_2=\sqrt{1^2+2^2}=\sqrt{5},\qquad
\|b\|_2=\sqrt{2^2+(-1)^2}=\sqrt{5}.
$$

The cosine similarity is $0/(\sqrt5\sqrt5)=0$, so the vectors are orthogonal.

### Solution 2

$Xw$ has shape $(3\times2)(2\times1)=3\times1$. The bias is a scalar broadcast to all three predictions, so $\hat y$ also has shape $(3,1)$. In code, explicitly check that the three-row batch remains the first axis; accidental transposes can produce a plausible but incorrect shape.

### Solution 3

$$
f'(w)=w-3,\qquad f'(0)=-3,\qquad
w_1=0-0.2(-3)=0.6.
$$

The old loss is $\frac12(0-3)^2=4.5$. The new loss is $\frac12(0.6-3)^2=2.88$. This step decreases the objective.

### Solution 4

At $(w,b)=(0,0)$, both predictions are zero and errors $\hat y-y$ are $-1$ and $-3$. Therefore:

$$
J=\frac{1}{4}(1+9)=2.5.
$$

For this objective,

$$
\frac{\partial J}{\partial w}=\frac1m\sum_i(\hat y_i-y_i)x_i=-1.5,
\qquad
\frac{\partial J}{\partial b}=\frac1m\sum_i(\hat y_i-y_i)=-2.
$$

One update gives $w_1=0-0.1(-1.5)=0.15$ and $b_1=0-0.1(-2)=0.2$.

### Solution 5

The derivative of $\frac{\lambda}{2}w^2$ is $\lambda w$. The regularized gradient is $-1.5+0.1(2)=-1.3$. Penalizing an intercept can shift every prediction away from the target's natural baseline; most linear-model conventions regularize slopes but leave the intercept unpenalized.

### Solution 6

Let $D$ mean “has the condition” and $+$ mean “tests positive.”

$$
P(+)=P(+\mid D)P(D)+P(+\mid\neg D)P(\neg D)
=0.9(0.01)+0.05(0.99)=0.0585.
$$

Then:

$$
P(D\mid+)=\frac{P(+\mid D)P(D)}{P(+)}
=\frac{0.009}{0.0585}\approx0.154.
$$

Only about 15.4% of positive results correspond to the condition under these assumptions. The condition is rare, so false positives from the much larger healthy group are numerous relative to true positives.

### Solution 7

The mean is $3$. Squared deviations are $4,0,4$, whose sum is $8$. Population variance is $8/3$. Sample variance is $8/(3-1)=4$. The sample formula uses $n-1$ because the mean was estimated from the same sample.

### Solution 8

$$
\operatorname{SE}(\bar{x})\approx\frac{s}{\sqrt n}=\frac4{4}=1,
\qquad
12\pm1.96(1)=[10.04,13.96].
$$

With only 16 observations, strong skew or extreme outliers can make the normal approximation poor. A $t$ multiplier is usually used when estimating a mean with unknown variance, especially for small samples.

### Solution 9

The Bernoulli likelihood is proportional to $p^7(1-p)^3$, so $\hat p_{\mathrm{MLE}}=7/10=0.7$. Beta conjugacy gives posterior $\operatorname{Beta}(2+7,2+3)=\operatorname{Beta}(9,5)$. Its mode is:

$$
\hat p_{\mathrm{MAP}}=\frac{9-1}{9+5-2}=\frac8{12}\approx0.667.
$$

The prior pulls the estimate toward $0.5$. MAP is a posterior mode; the posterior mean would instead be $9/14\approx0.643$.

### Solution 10

$$
\ell(y,p)=-y\log p-(1-y)\log(1-p)=-\log(0.8)\approx0.223.
$$

When the true label is 1 and $p$ approaches zero, the loss grows without bound. The logarithm strongly penalizes confident wrong predictions.

### Solution 11

The mean is already zero. The population covariance is:

$$
\Sigma=\frac12\left(
\begin{bmatrix}1\\0\end{bmatrix}\begin{bmatrix}1&0\end{bmatrix}
+
\begin{bmatrix}-1\\0\end{bmatrix}\begin{bmatrix}-1&0\end{bmatrix}
\right)
=\begin{bmatrix}1&0\\0&0\end{bmatrix}.
$$

The eigenvalues are 1 and 0; an eigenvector for the largest eigenvalue is $(1,0)$. Projection onto it loses no variation in this toy dataset because every point has second coordinate zero. In real data, discarded components can contain task-relevant information even when their variance is small.

### Solution 12

$$
\frac{(2.001)^2-(1.999)^2}{0.002}
=\frac{4.004001-3.996001}{0.002}=4.
$$

The analytic derivative is $2w=4$. Finite differences are an approximation in general; the step size must balance truncation error against floating-point cancellation.

### Solution 13

$$
\nabla f(x,y)=\begin{bmatrix}2x\\6y\end{bmatrix},\qquad
\nabla^2 f(x,y)=\begin{bmatrix}2&0\\0&6\end{bmatrix}.
$$

Both eigenvalues of the Hessian are positive, so the function is strictly convex and has a unique global minimum at $(0,0)$.

### Solution 14

Using $H(p)=-\sum_i p_i\log_2 p_i$:

$$
H(0.5,0.5)=1\text{ bit},
\qquad
H(0.9,0.1)=-0.9\log_2(0.9)-0.1\log_2(0.1)\approx0.469\text{ bits}.
$$

The second variable is less uncertain because one outcome is much more likely.

## Mastery check

You are ready to move on when you can derive the regression gradient, explain the base-rate effect in Bayes' rule, distinguish MLE from MAP, and check a gradient numerically without copying the solutions.
