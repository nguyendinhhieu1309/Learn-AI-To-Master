# AI Mathematics Formula Cheatsheet

Use this as a quick reference after studying the relevant lesson. Each formula is followed by a plain-language reminder; symbols can mean different things in different fields, so check the local definitions.

## Linear algebra

Vector dot product and cosine similarity:

$$
x^\top y = \sum_{i=1}^{d}x_i y_i,
\qquad
\cos(x,y)=\frac{x^\top y}{\|x\|_2\,\|y\|_2}
$$

The dot product combines aligned coordinates; cosine similarity compares direction and ignores magnitude.

Matrix multiplication shapes:

$$
A\in\mathbb{R}^{m\times n},\quad
x\in\mathbb{R}^{n}
\quad\Longrightarrow\quad
Ax\in\mathbb{R}^{m}
$$

The inner dimensions must match. For a batch of row vectors $X\in\mathbb{R}^{b\times n}$ and weights $W\in\mathbb{R}^{n\times k}$, the output is $XW\in\mathbb{R}^{b\times k}$.

## Calculus and optimization

Gradient descent:

$$
\theta_{t+1}=\theta_t-\eta\nabla_\theta J(\theta_t)
$$

The gradient points toward steepest increase locally; subtract it to reduce the objective. The learning rate $\eta$ controls step size.

Chain rule:

$$
\frac{\partial L}{\partial x}
=\frac{\partial L}{\partial y}\frac{\partial y}{\partial x}
$$

Backpropagation applies the chain rule repeatedly through a computational graph.

Common activations and derivatives:

$$
\sigma(z)=\frac{1}{1+e^{-z}},
\qquad \sigma'(z)=\sigma(z)(1-\sigma(z)),
\qquad \operatorname{ReLU}(z)=\max(0,z)
$$

For ReLU, the derivative is $1$ when $z>0$ and $0$ when $z<0$; implementations choose a convention at $z=0$.

## Probability and statistics

Bayes’ rule:

$$
P(A\mid B)=\frac{P(B\mid A)P(A)}{P(B)}
$$

It updates a prior belief using the likelihood of observed evidence.

Expectation, variance, standard error:

$$
\mathbb{E}[X]=\sum_x xP(X=x),\qquad
\operatorname{Var}(X)=\mathbb{E}[(X-\mathbb{E}[X])^2],\qquad
SE(\bar{x})=\frac{s}{\sqrt{n}}
$$

Variance measures spread of individual values; standard error measures uncertainty in a sample mean under the sampling assumptions.

Bernoulli probability and log likelihood for binary observations:

$$
P(X=x)=p^x(1-p)^{1-x},
\qquad
\ell(p)=\sum_{i=1}^{N}\left[x_i\log p+(1-x_i)\log(1-p)\right]
$$

Under independent flips, the MLE is $\hat p=\frac{1}{N}\sum_i x_i$. The log converts a product of probabilities into a sum.

Maximum likelihood and MAP:

$$
\hat\theta_{\mathrm{MLE}}=\arg\min_\theta-\log p(D\mid\theta),
\qquad
\hat\theta_{\mathrm{MAP}}=\arg\min_\theta\left[-\log p(D\mid\theta)-\log p(\theta)\right]
$$

MAP adds a prior penalty; a zero-centered Gaussian prior on weights corresponds to an $L_2$ penalty.

## Supervised learning

Mean squared error:

$$
\operatorname{MSE}=\frac{1}{N}\sum_{i=1}^{N}(y_i-\hat{y}_i)^2
$$

Binary cross-entropy for predicted probability $p_i$ and binary label $y_i$:

$$
L=-\frac{1}{N}\sum_{i=1}^{N}
\left[y_i\log p_i+(1-y_i)\log(1-p_i)\right]
$$

Regularized empirical risk:

$$
J(\theta)=\frac{1}{N}\sum_{i=1}^{N}\ell(f_\theta(x_i),y_i)
+\lambda\Omega(\theta)
$$

The first term fits observed examples; the second discourages complexity according to a chosen penalty.

Linear regression, with $X\in\mathbb{R}^{N\times d}$ and residual $r=Xw+b\mathbf{1}-y$:

$$
J(w,b)=\frac{1}{2N}\|r\|_2^2,
\qquad
\nabla_wJ=\frac{1}{N}X^Tr,
\qquad
\frac{\partial J}{\partial b}=\frac{1}{N}\mathbf{1}^Tr
$$

Logistic regression with $p=\sigma(Xw+b\mathbf{1})$:

$$
\nabla_wJ=\frac{1}{N}X^T(p-y),
\qquad
\frac{\partial J}{\partial b}=\frac{1}{N}\mathbf{1}^T(p-y)
$$

k-means objective:

$$
J=\sum_{i=1}^{N}\|x_i-\mu_{c_i}\|_2^2
$$

Assign each point to the closest centroid, then replace each centroid with its cluster mean.

SVM hinge loss for $y_i\in\{-1,+1\}$ and score $s_i=w^Tx_i+b$:

$$
\ell_i=\max(0,1-y_is_i)
$$

The margin penalty is zero once a correctly classified example is at least one normalized margin from the boundary.

PCA explained variance retained by the first $k$ components:

$$
\operatorname{EVR}(k)=\frac{\sum_{j=1}^{k}\lambda_j}{\sum_{j=1}^{d}\lambda_j},
\qquad \lambda_j=\frac{\sigma_j^2}{N-1}
$$

Here $\lambda_j$ are covariance eigenvalues and $\sigma_j$ are singular values of centered data.

## Classification metrics

$$
\operatorname{Precision}=\frac{TP}{TP+FP},\qquad
\operatorname{Recall}=\frac{TP}{TP+FN},\qquad
F_1=2\frac{\operatorname{Precision}\cdot\operatorname{Recall}}
{\operatorname{Precision}+\operatorname{Recall}}
$$

Precision asks “how many predicted positives were correct?”; recall asks “how many actual positives were found?” Always define the positive class and report counts for imbalanced problems.

## Neural networks and attention

Softmax converts logits $z$ to a categorical distribution:

$$
\operatorname{softmax}(z_i)=\frac{e^{z_i}}{\sum_j e^{z_j}}
$$

For a one-hot target $y$, softmax plus cross-entropy has the simple logit gradient

$$
\frac{\partial L}{\partial z_i}=p_i-y_i
$$

where $p_i=\operatorname{softmax}(z)_i$. Framework cross-entropy functions often accept raw logits and compute this combination stably.

Scaled dot-product attention:

$$
\operatorname{Attention}(Q,K,V)
=\operatorname{softmax}\left(\frac{QK^\top}{\sqrt{d_k}}\right)V
$$

Queries select keys; the resulting weights mix value vectors. The scale $\sqrt{d_k}$ helps keep dot products in a numerically useful range.

Convolution output length for input $H$, kernel $K$, padding $P$, stride $S$, and dilation $D$:

$$
H_{\mathrm{out}}=\left\lfloor\frac{H+2P-D(K-1)-1}{S}\right\rfloor+1
$$

Apply the same formula to width. For a standard convolution with $C_{in}$ input channels, $C_{out}$ filters, and a $K_h\times K_w$ kernel, parameter count with biases is $C_{out}(C_{in}K_hK_w+1)$.

## Computer vision and 3D

Intersection over Union:

$$
\operatorname{IoU}(A,B)=\frac{|A\cap B|}{|A\cup B|}
$$

Pinhole camera projection (idealized):

$$
u=f_x\frac{X}{Z}+c_x,\qquad
v=f_y\frac{Y}{Z}+c_y
$$

Stereo depth for rectified cameras:

$$
Z=\frac{fB}{d}
$$

Here $f$ is focal length in pixels, $B$ is camera baseline, and $d$ is disparity in pixels. Units must be consistent.

## Reinforcement learning

Discounted return:

$$
G_t=\sum_{k=0}^{\infty}\gamma^k r_{t+k}
$$

Bellman optimality equation:

$$
Q^*(s,a)=\mathbb{E}\left[r+\gamma\max_{a'}Q^*(s',a')\mid s,a\right]
$$

The value of an action includes its immediate reward and discounted value of the next state.

## Notation and common errors

- $N$ often means number of examples; $d$ feature dimension; $b$ batch size; $\theta$ model parameters.
- Distinguish a probability $p$ from a logit $z$; cross-entropy implementations usually expect logits.
- State whether a metric is averaged over samples, classes, groups, or thresholds.
- Write units for physical quantities and coordinate frames for 3D points.
- A formula describes assumptions as well as a computation; independence, distribution, and sampling conditions matter.
