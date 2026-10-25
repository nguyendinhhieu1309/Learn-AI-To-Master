# Likelihood, Maximum Likelihood, and MAP

Likelihood turns a model's probability assumptions into a quantity we can fit from data. This is the bridge from probability to losses such as squared error and cross-entropy.

## Learning objectives

- Tell apart a probability distribution and a likelihood function.
- Derive the maximum-likelihood estimate for a Bernoulli probability.
- Explain why independent observations produce a sum of log-likelihoods.
- Connect common training losses to likelihood assumptions.
- Explain how a prior leads from maximum likelihood to MAP and regularization.

## 1. Probability and likelihood use the same expression differently

Suppose observations $D=(x_1,\ldots,x_n)$ are modeled using parameter $\theta$. The probability of the data under a fixed parameter is

$$
p(D\mid\theta).
$$

Once the data are observed and fixed, the same expression viewed as a function of $\theta$ is the **likelihood**:

$$
L(\theta;D)=p(D\mid\theta).
$$

Likelihood is not generally a probability distribution over $\theta$; its values need not sum or integrate to one. It tells us which parameter values make the observed data relatively plausible.

## 2. Maximum likelihood estimation (MLE)

MLE chooses the parameter that maximizes the likelihood:

$$
\hat{\theta}_{\mathrm{MLE}}=\arg\max_\theta p(D\mid\theta).
$$

For conditionally independent observations,

$$
p(D\mid\theta)=\prod_{i=1}^{n}p(x_i\mid\theta).
$$

Products of many small probabilities can underflow numerically. The logarithm is increasing, so it preserves the maximizer while converting products into sums:

$$
\hat{\theta}_{\mathrm{MLE}}
=\arg\max_\theta \sum_{i=1}^{n}\log p(x_i\mid\theta)
=\arg\min_\theta -\sum_{i=1}^{n}\log p(x_i\mid\theta).
$$

The negative log-likelihood (NLL) is therefore a natural loss. Dividing by $n$ changes the scale but not the minimizing parameter.

## 3. Worked example: estimate a coin's probability

Let $x_i=1$ for heads and $0$ for tails. Assume independent flips with unknown head probability $p$. The likelihood is

$$
L(p;D)=\prod_{i=1}^{n}p^{x_i}(1-p)^{1-x_i}
=p^{h}(1-p)^{n-h},
$$

where $h=\sum_i x_i$ is the number of heads. The log-likelihood is

$$
\ell(p)=h\log p+(n-h)\log(1-p).
$$

Differentiate and set to zero:

$$
\frac{d\ell}{dp}=\frac{h}{p}-\frac{n-h}{1-p}=0
\quad\Longrightarrow\quad
\hat p_{\mathrm{MLE}}=\frac{h}{n}.
$$

If 7 of 10 flips are heads, $\hat p=0.7$. With only 10 observations, that estimate is uncertain; MLE does not itself report how much evidence supports the estimate.

## 4. Why familiar ML losses appear

### Squared error is Gaussian NLL

Assume a regression target is a model prediction plus independent Gaussian noise with fixed variance $\sigma^2$:

$$
y_i\mid x_i,\theta\sim\mathcal{N}(f_\theta(x_i),\sigma^2).
$$

Ignoring terms that do not depend on $\theta$, the negative log-likelihood is proportional to

$$
\sum_i\left(y_i-f_\theta(x_i)\right)^2.
$$

Thus minimizing squared error corresponds to fitting a conditional-mean model under a Gaussian-noise assumption. If errors have heavy tails or outliers, that assumption may be a poor description; MAE or a robust likelihood may fit the problem better.

### Binary cross-entropy is Bernoulli NLL

For binary target $y_i\in\{0,1\}$ and predicted probability $p_i=P(y_i=1\mid x_i)$,

$$
P(y_i\mid x_i)=p_i^{y_i}(1-p_i)^{1-y_i}.
$$

Taking the negative logarithm gives binary cross-entropy:

$$
\ell_i=-\left[y_i\log p_i+(1-y_i)\log(1-p_i)\right].
$$

If $y_i=1$, this becomes $-\log p_i$; if $y_i=0$, it becomes $-\log(1-p_i)$. Confident predictions against the observed label receive a large penalty.

### Multiclass cross-entropy is categorical NLL

For class probabilities $p_{i1},\ldots,p_{iK}$ and one-hot label indicators $y_{ik}$,

$$
\ell_i=-\sum_{k=1}^{K}y_{ik}\log p_{ik}.
$$

Only the probability of the true class contributes when the label is one-hot. This is why a stable software loss usually accepts logits and integer labels directly.

## 5. MAP: likelihood plus prior information

Maximum a posteriori (MAP) estimation maximizes the posterior parameter probability:

$$
\hat{\theta}_{\mathrm{MAP}}
=\arg\max_\theta p(\theta\mid D)
=\arg\max_\theta p(D\mid\theta)p(\theta),
$$

where the last equality drops the normalizing factor $p(D)$, which does not depend on $\theta$. Taking negative logarithms:

$$
\hat{\theta}_{\mathrm{MAP}}
=\arg\min_\theta\left[-\log p(D\mid\theta)-\log p(\theta)\right].
$$

The prior adds a penalty:

- A zero-mean Gaussian prior on weights leads to an $L_2$ penalty, often called weight decay or ridge regularization.
- A zero-centered Laplace prior leads to an $L_1$ penalty, which can produce sparse weights.

The strength of the prior determines the penalty scale. This parameter must be selected without using the final test set.

## 6. A simple Beta prior example

For coin flips, choose a Beta prior $p\sim\operatorname{Beta}(\alpha,\beta)$. After $h$ heads and $t$ tails, the posterior is

$$
p\mid D\sim\operatorname{Beta}(\alpha+h,\beta+t).
$$

With a uniform prior $\operatorname{Beta}(1,1)$ and 7 heads in 10 flips, the posterior mean is

$$
\mathbb{E}[p\mid D]=\frac{1+7}{1+1+10}=\frac{8}{12}\approx0.667.
$$

This is pulled slightly toward the prior center 0.5. The prior can stabilize estimates with little data; a badly chosen prior can also distort them, so state and justify it.

## 7. Practice

1. A biased coin produces 12 heads in 20 flips. Find the MLE.
2. If an observation is very unlikely under every candidate model, how can its log-likelihood affect training?
3. Which noise assumption connects squared error with MLE?
4. If a model predicts probability 0.9 for a true positive label, calculate that example's binary cross-entropy.
5. In your own words, how does an $L_2$ penalty correspond to a Gaussian prior?

**Answer checks:** $0.6$; it gives a large negative log-likelihood and therefore a large loss; Gaussian conditional noise with fixed variance; $-\log(0.9)\approx0.105$; large weights are less probable under a zero-centered Gaussian prior, so optimization is discouraged from choosing them.

## Common mistakes

- Calling likelihood a probability distribution over model parameters.
- Maximizing probability products directly instead of using log probabilities.
- Forgetting that each loss encodes assumptions about targets or noise.
- Applying a prior or regularizer without explaining its scale or validating it.
