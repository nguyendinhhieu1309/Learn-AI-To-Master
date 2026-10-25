# Convex Optimization and Constraints

Machine-learning training is usually written as an optimization problem: choose parameters that minimize a loss while obeying practical or mathematical constraints. Convexity explains when a local solution is also globally optimal and why some classical models are easier to train than neural networks.

## Learning objectives

- Identify convex sets and convex functions from their definitions and examples.
- Relate gradients and Hessians to local minima and curvature.
- Explain why least squares and logistic loss have global optima, while neural networks generally do not.
- Write a constrained objective using a Lagrangian and interpret KKT conditions at a high level.
- Explain what regularization and constraints do to model fitting.

## 1. Convex sets

A set $C$ is convex if the straight line between any two points in the set remains inside it. For $x,y\in C$ and $\alpha\in[0,1]$:

$$
\alpha x+(1-\alpha)y\in C.
$$

Examples include an interval, a line, a filled circle, a half-space, and the set of positive semidefinite matrices. A circle's circumference without its interior is not a convex set: a chord between two boundary points passes through excluded interior points.

## 2. Convex functions

A function $f$ is convex on a convex domain when its graph lies at or below the straight chord between any two points:

$$
f(\alpha x+(1-\alpha)y)
\leq \alpha f(x)+(1-\alpha)f(y)
\quad\forall x,y,\;\alpha\in[0,1].
$$

For a differentiable one-dimensional function, a nonnegative second derivative everywhere is sufficient for convexity. For a twice-differentiable function of a vector, a positive-semidefinite Hessian everywhere is sufficient.

Examples:

- $x^2$ is convex; every local minimum is also a global minimum.
- $|x|$ is convex but not differentiable at zero.
- $e^x$ is convex.
- A linear function is both convex and concave.
- $\sin x$ is not convex over all real numbers because its curvature changes sign.

For a convex, differentiable function, any point with zero gradient is a global minimum. A flat region can contain many global minimizers. Convexity does not necessarily mean the solution is unique.

## 3. Curvature and the Hessian

For scalar $f(x)$, the derivative gives slope and the second derivative gives curvature. For parameter vector $\theta$, the Hessian collects second derivatives:

$$
H_{ij}=\frac{\partial^2J}{\partial\theta_i\partial\theta_j}.
$$

Positive-semidefinite Hessian means locally no direction bends downward; positive-definite Hessian means strictly positive curvature in every nonzero direction. A poorly conditioned Hessian has very different curvature scales, making gradient descent zig-zag unless step sizes are chosen carefully.

## 4. Why squared-error linear regression is convex

For design matrix $X$ and target $y$, least-squares loss is

$$
J(w)=\frac12\|Xw-y\|_2^2.
$$

Its gradient and Hessian are

$$
\nabla J(w)=X^T(Xw-y),
\qquad
\nabla^2J(w)=X^TX.
$$

For every vector $v$,

$$
v^TX^TXv=\|Xv\|_2^2\geq0,
$$

so the Hessian is positive semidefinite and the objective is convex. If $X$ has full column rank, $X^TX$ is positive definite and the solution is unique. If columns are dependent, there may be many coefficient vectors with the same predictions. Ridge adds positive curvature and can make the solution unique when $\lambda>0$.

Logistic regression with convex log-loss is also convex in its linear parameters. SVM hinge-loss optimization is convex. In these models, a reliable optimizer can find a global optimum of the stated objective, although data quality and objective choice still determine whether that solution is useful.

## 5. Why deep-network losses are generally non-convex

A neural network's output depends on products and compositions of layer parameters. Changing two layers together can produce valleys, saddles, symmetries, and many equivalent parameter settings. Its loss is generally non-convex, so a stationary point is not guaranteed to be a global minimum.

This does not mean training is hopeless. Large networks often have many solutions with useful training behavior; initialization, architecture, data, regularization, and optimization all affect which solution is reached. Validation performance and robustness matter more than a theoretical guarantee that the global minimum was found.

## 6. Constrained optimization and Lagrangians

Some problems minimize an objective subject to inequalities and equalities:

$$
\min_\theta f(\theta)
\quad\text{subject to}\quad g_i(\theta)\leq0,\;h_j(\theta)=0.
$$

The Lagrangian combines the objective and constraints with multipliers:

$$
\mathcal{L}(\theta,\alpha,\nu)
=f(\theta)+\sum_i\alpha_i g_i(\theta)+\sum_j\nu_jh_j(\theta),
\qquad \alpha_i\geq0.
$$

For a convex problem with suitable regularity, a solution satisfies the Karush–Kuhn–Tucker (KKT) conditions:

1. **Primal feasibility:** the original constraints hold.
2. **Dual feasibility:** inequality multipliers satisfy $\alpha_i\geq0$.
3. **Stationarity:** the Lagrangian gradient with respect to parameters is zero.
4. **Complementary slackness:** $\alpha_i g_i(\theta)=0$ for each inequality.

Complementary slackness says a constraint that is not active has multiplier zero; a constraint that affects the solution may have a positive multiplier.

For hard-margin SVM, the margin constraints are $y_i(w^Tx_i+b)\geq1$. Introducing multipliers leads to a dual problem whose solution depends on dot products between examples. Replacing those dot products with a kernel is what enables nonlinear SVM boundaries. The full dual derivation is optional at first; the geometric and soft-margin lessons come before it.

## 7. Regularization as a prior or constraint

Penalized and constrained forms are related:

$$
\min_\theta L(\theta)+\lambda\|\theta\|_2^2
\qquad\leftrightarrow\qquad
\min_\theta L(\theta)\;\text{subject to}\;\|\theta\|_2\leq c.
$$

The parameter $\lambda$ (or radius $c$) controls the tradeoff between fitting data and limiting parameter magnitude. The exact correspondence depends on the problem and constraint activity. Regularization is not merely a trick to lower training loss; its purpose is to improve the chosen objective's behavior on new data.

## 8. Gradient descent and Newton's method

Gradient descent uses first-order local information:

$$
\theta_{t+1}=\theta_t-\eta\nabla J(\theta_t).
$$

For a smooth convex objective, a suitably small step size decreases loss; with stronger assumptions, one can bound how quickly it approaches the optimum. A step size that ignores curvature can still diverge on a convex problem.

Newton's method uses curvature to scale the step:

$$
\theta_{t+1}=\theta_t-[\nabla^2J(\theta_t)]^{-1}\nabla J(\theta_t).
$$

It can converge quickly near a well-behaved optimum, but computing/storing the Hessian and solving the linear system can be prohibitive for large models. Deep learning therefore commonly uses stochastic first-order optimizers and variants rather than full Newton updates.

## Practice with answer checks

1. Is $f(x)=3x+2$ convex? **Answer:** yes; it is linear.
2. Why does $X^TX$ being positive semidefinite prove least-squares convexity? **Answer:** it is the loss Hessian and its quadratic form is $\|Xv\|^2\geq0$ for every direction $v$.
3. What does $\alpha_i g_i(\theta)=0$ express? **Answer:** an inactive inequality constraint has zero multiplier; a positive multiplier can occur only when the constraint is active.
4. Does a convex objective guarantee that a too-large gradient-descent step will work? **Answer:** no; the step still must account for curvature/smoothness.
