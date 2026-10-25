# Statistical Inference and Experiment Design

Statistics helps us distinguish a real pattern from variation caused by sampling. In AI work, it is used to compare models, estimate uncertainty, design data collection, and decide whether a change is safe to ship.

## 1. Population, sample, and estimand

- **Population**: the set of cases we want to make claims about, such as all future support tickets.
- **Sample**: the cases we actually observed.
- **Parameter**: a property of the population, usually unknown.
- **Statistic**: a quantity computed from the sample.
- **Estimand**: the precise quantity the experiment is intended to estimate, such as the change in resolution rate caused by a new model.

Before collecting data, specify the unit of analysis, target population, outcome, comparison, and time window. A test with one row per message answers a different question from one with one row per customer.

For independent observations with sample mean $\bar{x}$ and sample standard deviation $s$, the standard error of the mean is:

$$
SE(\bar{x}) = \frac{s}{\sqrt{n}}
$$

The standard error describes uncertainty in the estimate; it is not the spread of individual observations. Increasing the sample size by a factor of four halves this standard error, assuming the sampling process stays comparable.

## 2. Confidence intervals

A 95% confidence procedure has 95% long-run coverage under its assumptions. It does **not** mean that, after seeing one interval, there is a 95% probability that the fixed parameter lies inside it.

For a sufficiently large sample, a basic interval for a mean is:

$$
\bar{x} \pm 1.96\,SE(\bar{x})
$$

For small samples with approximately normal data, use a $t$ critical value instead of 1.96. For proportions, use Wilson or another well-behaved binomial interval rather than the naive normal interval when counts are small or the rate is near 0 or 1.

**Worked example.** A classifier correctly handles 820 of 1,000 independent cases. Its observed accuracy is $0.82$. The rough standard error is $\sqrt{0.82(0.18)/1000} \approx 0.012$. A rough 95% interval is about $0.82 \pm 0.024$, or 79.6%–84.4%. This uncertainty is large enough that a measured one-point gain would not be convincing by itself.

## 3. Hypothesis tests and practical significance

Set the question before inspecting results:

- Null hypothesis $H_0$: no effect, or a specified baseline effect.
- Alternative hypothesis $H_1$: an effect in the direction or range of interest.
- Test statistic: a normalized measure of how far the observed result is from what $H_0$ predicts.
- $p$-value: probability, assuming $H_0$ and the model assumptions, of data at least as incompatible with $H_0$ as the observed data.

A small $p$-value is not the probability that $H_0$ is true, and it does not measure the size or value of an effect. Always report an effect estimate and interval, then ask whether the change is useful given latency, compute, and error costs.

For two independent proportions $p_1$ and $p_2$, a rough standard error for their difference is:

$$
SE(p_1-p_2) \approx \sqrt{\frac{p_1(1-p_1)}{n_1} + \frac{p_2(1-p_2)}{n_2}}
$$

The independent-sample formula is inappropriate for paired data. If the same cases are scored by both systems, analyze the paired differences or use a paired bootstrap.

## 4. Bootstrap and paired comparisons

The bootstrap approximates the sampling distribution by repeatedly sampling rows **with replacement** from the observed sample and recomputing a statistic. For model comparison, resample the same examples for both systems, preserving each pair. This estimates uncertainty in the difference, not in two unrelated scores.

```text
observed_difference = metric(model_B, examples) - metric(model_A, examples)
for b in 1..B:
    sample = resample_rows_with_replacement(examples)
    diff[b] = metric(model_B, sample) - metric(model_A, sample)
interval = percentile(diff, [2.5, 97.5])
```

The bootstrap cannot fix a biased or unrepresentative dataset. Resample at the true independent unit: for repeated records from a user, resample users rather than rows.

## 5. Multiple comparisons and repeated peeking

Trying many models, thresholds, prompts, or slices and reporting only the winner inflates the chance of a false discovery. Keep a final holdout set untouched during selection; use validation or nested cross-validation for tuning. In formal experiments, predefine primary metrics and use a correction or hierarchical testing plan when many hypotheses matter.

Repeatedly checking a fixed-horizon $p$-value and stopping as soon as it falls below a threshold also changes the false-positive rate. Use a sequential testing procedure or decide sample size and stopping rules in advance.

## 6. A/B tests for AI systems

Randomize the unit that can interfere with other units. For example, assign a customer consistently to one system if conversations within a customer affect one another. Track guardrails as well as the primary outcome: failure rate, escalation, latency, cost, and subgroup outcomes.

Watch for:

- **Sample-ratio mismatch**: allocation differs from the planned randomization, suggesting instrumentation or assignment bugs.
- **Novelty and time effects**: users or traffic change behavior after launch.
- **Interference**: one user’s treatment affects another user’s outcome.
- **Logging gaps**: a model may appear better because difficult cases were not recorded.
- **Selection bias**: evaluating only cases where a model returned a confident answer hides abstentions and failures.

## 7. Exercises

1. A model improves from 91.0% to 91.5% on a test set of 400 examples. Why is the point estimate insufficient to claim a reliable gain?
2. The same 2,000 records are scored by two models. Why can a paired bootstrap be more informative than treating the scores as independent?
3. A team tries 80 prompt variants on the test set and selects the best. What has happened to the test set, and how should the next evaluation be designed?
4. An A/B test randomizes messages, but outcomes from the same user are correlated. What should be randomized or resampled instead?

## 8. Check your understanding

1. With only point estimate and sample size, you still need a sampling model or resampling scheme to estimate uncertainty. A 0.5-point gain can easily be sampling noise.
2. Pairing removes variation shared by both models on easy or hard examples and directly estimates per-case differences.
3. The test set has become part of model selection and is no longer an unbiased final estimate. Collect a new holdout or use a locked external evaluation set.
4. Randomize and analyze at the user level, or use a cluster-aware design that accounts for correlation within a user.

## References

- [NIST/SEMATECH e-Handbook of Statistical Methods](https://www.itl.nist.gov/div898/handbook/)
- [Efron and Tibshirani, An Introduction to the Bootstrap](https://doi.org/10.1007/978-1-4899-4541-9)
