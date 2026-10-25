# Probability Distributions and Bayes

This lesson builds the probability tools used by classifiers, uncertainty estimates, generative models, and statistical learning. Each formula names what is random, what is observed, and what is being conditioned on.

## Learning objectives

By the end, you should be able to:

- distinguish a random variable from one observed value;
- compute joint, marginal, and conditional probabilities;
- use Bayes' rule without reversing the condition;
- recognize Bernoulli, categorical, binomial, and Gaussian distributions;
- connect expected value and variance to data summaries and model outputs.

## 1. Events, random variables, and distributions

An **event** is a set of possible outcomes, such as “the coin lands heads.” A **random variable** maps each outcome to a number. Capital $X$ usually names the random variable; lowercase $x$ is one value it may take. A **distribution** assigns probabilities or probability density to the possible values of $X$.

For a discrete random variable, probabilities sum to one:

$$
\sum_x P(X=x)=1.
$$

For a continuous variable, probabilities are represented by a density $p(x)$ and integrate to one. A density value itself is not a probability; the probability of an interval is the area under the density on that interval.

## 2. Joint, marginal, conditional, and independent events

The joint probability $P(X=x,Y=y)$ describes two events together. The marginal probability of $X$ is found by adding out the possible values of $Y$:

$$
P(X=x)=\sum_y P(X=x,Y=y).
$$

For events $A$ and $B$, conditional probability is

$$
P(A\mid B)=\frac{P(A\cap B)}{P(B)}, \qquad P(B)>0.
$$

Independence means learning one event does not change the probability of the other:

$$
P(A\cap B)=P(A)P(B), \qquad P(A\mid B)=P(A).
$$

Independence is an assumption, not a synonym for “different columns.” Two measurements from the same person, adjacent frames in a video, and repeated events from one customer are often dependent.

### Worked example: a medical test and a rare condition

Suppose 1% of a population has a condition. A test has 90% sensitivity and a 5% false-positive rate. Imagine 10,000 people:

- 100 have the condition; about 90 test positive.
- 9,900 do not; about 495 test positive by mistake.
- Of the 585 positive tests, 90 are true positives.

Therefore,

$$
P(\text{condition}\mid +)=\frac{90}{90+495}\approx 0.154.
$$

The test's 90% sensitivity is $P(+\mid\text{condition})$. It is not $P(\text{condition}\mid +)$. The base rate changes the answer.

## 3. Bayes' rule

Bayes' rule updates a prior belief after observing evidence:

$$
P(A\mid B)=\frac{P(B\mid A)P(A)}{P(B)}.
$$

Here $P(A)$ is the prior, $P(B\mid A)$ is the likelihood of the evidence under $A$, and $P(A\mid B)$ is the posterior. For mutually exclusive possibilities $A_1,\ldots,A_K$,

$$
P(B)=\sum_{k=1}^{K}P(B\mid A_k)P(A_k).
$$

Naive Bayes uses Bayes' rule and approximates a feature vector's class-conditional likelihood as a product of feature likelihoods. The conditional-independence assumption is often false, but the resulting classifier can still be a strong, fast baseline.

## 4. Expectation and variance

The expectation is the probability-weighted average outcome:

$$
\mathbb{E}[X]=\sum_x xP(X=x)
$$

for a discrete variable, or $\mathbb{E}[X]=\int x p(x)\,dx$ for a continuous one. Variance measures mean squared distance from the expected value:

$$
\operatorname{Var}(X)=\mathbb{E}\left[(X-\mathbb{E}[X])^2\right]
=\mathbb{E}[X^2]-\mathbb{E}[X]^2.
$$

The standard deviation is $\sigma=\sqrt{\operatorname{Var}(X)}$ and has the same units as $X$. For a constant $a$ and random variables $X,Y$:

$$
\mathbb{E}[aX+b]=a\mathbb{E}[X]+b,
\qquad
\operatorname{Var}(aX+b)=a^2\operatorname{Var}(X).
$$

Expected value is a theoretical average under a distribution; the sample mean $\bar{x}$ is an estimate from observed data.

## 5. Common distributions

### Bernoulli: one yes/no outcome

If $X\in\{0,1\}$ and $P(X=1)=p$, then $X\sim\operatorname{Bernoulli}(p)$:

$$
P(X=x)=p^x(1-p)^{1-x}, \qquad
\mathbb{E}[X]=p, \qquad
\operatorname{Var}(X)=p(1-p).
$$

This is the basic distribution for one binary label or one coin flip.

### Categorical: one of several classes

For $K$ classes with probabilities $p_1,\ldots,p_K$, each $p_k\geq 0$ and $\sum_kp_k=1$. A one-hot target has one entry equal to one; its categorical probability is the entry selected by the true class. A softmax model produces a categorical probability vector from $K$ logits.

### Binomial: number of successes in repeated trials

For $n$ independent Bernoulli trials with the same success probability $p$, their sum $S$ follows $\operatorname{Binomial}(n,p)$:

$$
P(S=k)=\binom{n}{k}p^k(1-p)^{n-k},
\qquad
\mathbb{E}[S]=np,
\qquad
\operatorname{Var}(S)=np(1-p).
$$

The assumptions matter: trials must be independent and share the same $p$.

### Gaussian (normal): a bell-shaped continuous distribution

For a one-dimensional Gaussian with mean $\mu$ and variance $\sigma^2$:

$$
p(x)=\frac{1}{\sqrt{2\pi\sigma^2}}
\exp\!\left(-\frac{(x-\mu)^2}{2\sigma^2}\right).
$$

The mean sets the center; the variance controls spread. Gaussian noise leads to squared-error loss when fitting a model by maximum likelihood. A multivariate Gaussian uses a mean vector and covariance matrix $\Sigma$; the off-diagonal entries describe linear relationships between dimensions.

## 6. Law of large numbers and sampling

For independent, identically distributed samples with finite mean, the sample average tends to the true expectation as sample count grows:

$$
\bar{X}_n=\frac{1}{n}\sum_{i=1}^n X_i \longrightarrow \mathbb{E}[X].
$$

This does not fix biased sampling, distribution shift, or dependence between examples. A million examples from the wrong population can estimate the wrong quantity very precisely.

## 7. Practice

1. A fair die is rolled. Write $P(X=3)$ and $\mathbb{E}[X]$.
2. If $P(A)=0.2$, $P(B\mid A)=0.5$, and $P(B\mid \neg A)=0.1$, calculate $P(A\mid B)$.
3. A binary model outputs $p=0.7$. What are the probabilities it assigns to class 1 and class 0? What distribution describes this single label?
4. Explain why $\operatorname{Var}(2X+3)=4\operatorname{Var}(X)$.

**Answer checks:** $1/6$ and $3.5$; $0.2(0.5)/(0.2(0.5)+0.8(0.1))=0.556$; Bernoulli with probabilities 0.7 and 0.3; adding a constant does not affect spread and multiplying values by two doubles deviations, so squared deviations quadruple.

## Common mistakes

- Reversing $P(A\mid B)$ and $P(B\mid A)$.
- Calling a probability density a probability.
- Assuming feature columns are independent because they have different names.
- Reporting an average without uncertainty, spread, or a description of the sample.
