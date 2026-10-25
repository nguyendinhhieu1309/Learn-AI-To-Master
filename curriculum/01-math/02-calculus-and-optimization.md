# Calculus, Gradients, and Optimization

## Learning objectives

Understand a derivative as a local rate of change, a gradient as a direction of change, and gradient descent as repeated steps toward lower loss.

## 1. Derivative: how fast does a value change?

For a function $f(x)$, the derivative $f'(x)$ tells us the slope near $x$. A line $f(x)=3x+1$ has derivative 3 everywhere: increasing $x$ by a small amount changes $f$ by about three times that amount. For $f(x)=x^2$, the derivative is $2x$; at $x=3$, the slope is 6.

The derivative is local. For a large step, the slope can change, so the linear approximation may be poor.

## 2. Partial derivatives and gradients

A model has many parameters. A partial derivative changes one parameter while holding the others fixed. The **gradient** collects all partial derivatives:

$$
\nabla_\theta J(\theta) = \left[\frac{\partial J}{\partial \theta_1}, ..., \frac{\partial J}{\partial \theta_d}\right].
$$

The gradient points toward locally increasing $J$. To reduce a loss, move in the opposite direction.

## 3. Gradient descent by hand

Minimize $J(w)=(w-3)^2$. Its derivative is $2(w-3)$. Starting at $w=0$ with learning rate $\eta=0.1$, the update is

$$
w_{\mathrm{new}}=w-\eta J'(w)=0-0.1(-6)=0.6.
$$

Repeating the update moves `w` toward 3 and reduces the loss. If the learning rate is too small, progress is slow. If too large, updates can bounce over the minimum or diverge.

## 4. Chain rule and neural networks

If $y=f(g(x))$, then

$$
\frac{dy}{dx}=f'(g(x))g'(x).
$$

This chain rule lets us compute how an output changes through a sequence of layers. Backpropagation applies it efficiently to every parameter in a neural network.

The chain rule does not mean gradients always flow well: repeated multiplication can make them tiny (vanishing gradients) or huge (exploding gradients). Initialization, architecture, normalization, and optimizer settings can help.

## 5. Loss and regularization

A **loss** measures prediction error. Mean squared error is

$$
\operatorname{MSE}=\frac{1}{n}\sum_{i=1}^{n}(y_i-\hat y_i)^2.
$$

For classification, cross-entropy penalizes low probability assigned to the true class. The optimizer minimizes average training loss.

Regularization adds a penalty, for example

$$
J_{\mathrm{total}}=J_{\mathrm{data}}+\lambda\|w\|_2^2.
$$

It discourages overly large weights. The penalty coefficient $\lambda$ is a hyperparameter; tune it using validation data.

## 6. A practical optimization loop

1. Make a forward prediction.
2. Compare predictions with labels to compute loss.
3. Compute gradients of loss with respect to parameters.
4. Update parameters with the optimizer.
5. Repeat over batches; measure validation performance separately.

Inspect a loss curve. Training loss falling while validation loss rises is a warning for overfitting. Both losses staying high can indicate a model that is too simple, bad inputs, poor labels, or an optimization problem.

## Check your understanding

For $J(w)=(w-2)^2$, at $w=5$, the gradient is 6. With learning rate $\eta=0.1$, the next value is $5-0.1(6)=4.4$. Check that it moved toward 2. This one-step check is a useful way to catch a reversed sign.

## Common mistakes

- Updating in the direction of the gradient when minimizing.
- Assuming a smaller training loss always means a better generalizing model.
- Changing the learning rate, model, and data at the same time, making the cause unclear.
- Forgetting that gradients depend on the current data batch and parameter values.

## A deeper derivation: gradients for linear regression

The earlier sections introduced the gradient-descent idea. This section derives the update used by a real model so the symbols and signs are not a memorized rule.

Assume there are $n$ examples and one feature. Example $i$ has input $x_i$, target $y_i$, predicted value $\hat y_i=wx_i+b$, and residual $r_i=\hat y_i-y_i$. Use half mean squared error:

$$
J(w,b)=\frac{1}{2n}\sum_{i=1}^{n}(wx_i+b-y_i)^2.
$$

The factor $1/2$ cancels the factor 2 created by differentiating a square. Applying the chain rule to one term gives

$$
\frac{\partial J}{\partial w}
=\frac{1}{n}\sum_{i=1}^{n}(wx_i+b-y_i)x_i,
\qquad
\frac{\partial J}{\partial b}
=\frac{1}{n}\sum_{i=1}^{n}(wx_i+b-y_i).
$$

Why does $x_i$ appear in the weight gradient? Increasing $w$ changes the prediction by $x_i$ units; increasing $b$ changes every prediction by one unit.

### Worked update with one training example

Let $x=2$, $y=5$, $w=1$, and $b=0$. The prediction is $\hat y=2$ and the half-squared loss is $J=\tfrac12(2-5)^2=4.5$. The gradients are

$$
\frac{\partial J}{\partial w}=(2-5)\cdot2=-6,
\qquad
\frac{\partial J}{\partial b}=2-5=-3.
$$

With learning rate $\eta=0.1$, update both parameters using the **old** gradients:

$$
w_{\mathrm{new}}=1-0.1(-6)=1.6,
\qquad
b_{\mathrm{new}}=0-0.1(-3)=0.3.
$$

The new prediction is $2(1.6)+0.3=3.5$ and the new loss is $\tfrac12(3.5-5)^2=1.125$. The loss fell, which is a useful sign check. One example is not enough to show that this update will generalize to new data.

## From scalar examples to matrix gradients

For $n$ examples and $d$ features, put one example in each row of $X\in\mathbb{R}^{n\times d}$, use $w\in\mathbb{R}^{d}$ and $y\in\mathbb{R}^{n}$. The prediction and residual vectors are

$$
\hat y=Xw+b\mathbf{1},
\qquad
r=Xw+b\mathbf{1}-y.
$$

The half mean squared error and gradients are

$$
J(w,b)=\frac{1}{2n}r^Tr,
\qquad
\nabla_w J=\frac{1}{n}X^Tr,
\qquad
\frac{\partial J}{\partial b}=\frac{1}{n}\mathbf{1}^Tr.
$$

Check the dimensions before multiplying:

$$
(n\times d)^T(n\times1)=(d\times n)(n\times1)=(d\times1),
$$

which matches the number of weights. For multiple outputs, the derivative becomes a matrix with the same shape as the weight matrix. Shape checking is a simple way to catch many calculus and code errors.

## Derivative rules worth knowing

For scalars $a,b$ and differentiable functions $f,g$:

$$
\frac{d}{dx}c=0,
\qquad
\frac{d}{dx}x^k=kx^{k-1},
\qquad
\frac{d}{dx}e^x=e^x,
\qquad
\frac{d}{dx}\ln x=\frac{1}{x},
$$

$$
\frac{d}{dx}[af(x)+bg(x)]=af'(x)+bg'(x),
\qquad
\frac{d}{dx}f(g(x))=f'(g(x))g'(x).
$$

The last identity is the chain rule. For sigmoid $\sigma(z)=1/(1+e^{-z})$,

$$
\sigma'(z)=\sigma(z)(1-\sigma(z)).
$$

For a vector-valued function, derivatives form a **Jacobian**: rows correspond to outputs and columns to inputs. A scalar loss with vector parameters has a gradient with one component per parameter. State which convention you use; transposes differ between row-gradient and column-gradient conventions.

## Numerical gradient check

If an analytic gradient is difficult to trust, compare one component with a central finite difference:

$$
\frac{\partial J}{\partial w_j}\approx
\frac{J(w+\epsilon e_j)-J(w-\epsilon e_j)}{2\epsilon},
$$

where $e_j$ changes only component $j$ and $\epsilon$ is small. On a tiny problem in double precision, the numerical and analytic values should be close. This check is for debugging a custom derivative; finite differences are too expensive and imprecise to train large neural networks.

## What can make gradient descent fail?

- **Learning rate too large:** the update can jump across the valley and increase loss. Reduce it and inspect a loss-versus-step plot.
- **Learning rate too small:** the loss decreases very slowly. Try a larger rate on a safe, small experiment.
- **Features on very different scales:** the loss surface may be narrow in one direction and wide in another. Fit scaling parameters on the training split and reuse them at validation and inference.
- **Bad gradients or data:** check finite values, labels, tensor shapes, and a tiny-batch overfit test before changing the architecture.
- **Non-convex objective:** neural-network loss can have many flat regions and saddle points; a zero gradient does not guarantee a global minimum.

Batch gradient descent uses all examples for each update. Stochastic gradient descent uses one example, and mini-batch gradient descent uses a small batch. Mini-batches trade a noisy gradient estimate for faster, more frequent updates. Momentum and adaptive methods modify the update history or coordinate scales; they still depend on sensible data, gradients, and learning-rate choices.

## Practice with answers

1. For $J(w)=\tfrac12(3w-6)^2$, calculate $dJ/dw$ at $w=1$. **Answer:** $3w-6=-3$, so $dJ/dw=(3w-6)\cdot3=-9$.
2. With $\eta=0.1$, what is the next $w$? **Answer:** $1-0.1(-9)=1.9$, closer to the minimum $w=2$.
3. If $X$ has shape $(100,8)$ and $r$ has shape $(100,)$, what is the shape of $X^Tr$? **Answer:** $(8,)$; there is one gradient per feature.
4. What is the main purpose of a numerical gradient check? **Answer:** catch an implementation or chain-rule error on a small problem.
