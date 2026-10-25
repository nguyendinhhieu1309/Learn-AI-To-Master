# Probability and Statistics

## Learning objectives

Use probability to describe uncertainty, statistics to summarize observed data, and samples to reason cautiously about a larger population.

## 1. Probability and events

Probability ranges from 0 (impossible) to 1 (certain). For a fair six-sided die, $P(X=4)=1/6$. If events cannot happen together, $P(A\cup B)=P(A)+P(B)$. In general,

$$
P(A \cup B)=P(A)+P(B)-P(A \cap B).
$$

The subtraction avoids counting outcomes in both events twice.

## 2. Conditional probability and Bayes' rule

The conditional probability $P(A\mid B)$ is the probability of event $A$ given that $B$ is known. Bayes' rule reverses a conditional:

$$
P(A\mid B)=\frac{P(B\mid A)P(A)}{P(B)}.
$$

Suppose a rare condition affects 1% of people. A test is 90% sensitive ($P(+\mid C)=0.9$) and has a 5% false-positive rate ($P(+\mid\neg C)=0.05$). Out of 10,000 people, about 90 true positives and 495 false positives are expected. The chance of having the condition given a positive test is

$$
P(C\mid+)=\frac{90}{90+495}\approx0.154,
$$

about 15.4%, not 90%. Base rates matter.

## 3. Random variables, expectation, and variance

A random variable represents an uncertain numerical outcome. Its expectation is the long-run average. For a discrete random variable:

$$
\mathbb{E}[X]=\sum_x xP(X=x).
$$

Variance measures spread around the mean, and standard deviation is its square root:

$$
\operatorname{Var}(X)=\mathbb{E}\left[(X-\mathbb{E}[X])^2\right],
\qquad \sigma_X=\sqrt{\operatorname{Var}(X)}.
$$

Two datasets can have the same mean but very different spread. Models and metrics should not be summarized by an average alone when variability matters.

## 4. Samples and estimates

A dataset is usually a sample, not the whole population we care about. Sample means vary from sample to sample. More representative, independent observations generally make estimates more stable; a large biased sample can still be misleading.

A confidence interval describes a procedure that, across repeated samples under assumptions, covers the true parameter at a stated rate. It is not a probability statement that a fixed parameter moves around after the interval is computed.

For independent observations, the standard error of a sample mean estimates how much the mean would vary across repeated samples:

$$
SE(\bar{x})=\frac{s}{\sqrt{n}}.
$$

For a sufficiently large sample, a rough 95% interval for a mean is $\bar{x}\pm1.96SE(\bar{x})$. For small samples or proportions near 0 or 1, use an interval method whose assumptions match the data rather than applying this approximation mechanically.

## 5. Association is not causation

Correlation describes how two quantities vary together. It can arise because one causes the other, because both share another cause, through selection effects, or by coincidence. A predictive model can use a correlated feature without establishing a causal effect.

## 6. ML evaluation and uncertainty

Predictions and scores are uncertain because data are limited and deployment data may differ. Use a held-out test set for a final estimate, and consider repeated splits or bootstrap intervals when sample size allows. Avoid repeatedly choosing models based on the test set; that makes it part of training-by-selection.

## Check your understanding

1. A disease occurs in 1 of 100 people. Why can a positive result from a fairly accurate test still be wrong? **Answer:** false positives among the much larger unaffected group can outnumber true positives.
2. Does correlation prove that one feature causes a prediction target? **Answer:** no.
3. If data are sampled from only one hospital, what may fail elsewhere? **Answer:** representativeness and distribution assumptions.

## Common mistakes

- Confusing $P(A\mid B)$ with $P(B\mid A)$.
- Treating a sample statistic as the exact population value.
- Reporting only the mean while hiding spread or subgroup differences.
- Interpreting a predictive association as a causal claim.
