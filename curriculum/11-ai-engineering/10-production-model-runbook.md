# Production Model Runbook: Serving, Monitoring, and Rollback

A trained model is not a production system. The deployed behavior also depends on preprocessing, input contracts, model artifacts, infrastructure, traffic, monitoring, and operational decisions.

## Learning objectives

- Package a model with its preprocessing and input/output contract.
- Select batch, online, or asynchronous serving from the product requirement.
- Budget latency and cost using measurable components.
- Separate service health from model quality.
- Define safe rollout, incident response, and rollback conditions.

## Prerequisites

Complete [AI Engineering](README.md), especially [model serving](05-model-serving-and-inference-patterns.md), [reliability](07-reliability-observability-and-cost.md), and [inference latency and cost](09-inference-serving-latency-and-cost.md).

## 1. Package the prediction contract

A release should identify more than a weight file. Record:

- model artifact and checksum;
- preprocessing and feature schema version;
- training data snapshot or reference;
- evaluation report and intended operating point;
- dependency/runtime versions;
- supported input range and missing-value behavior;
- owner, creation time, and rollback target.

The model and preprocessing must be loaded as one versioned unit. If a new model expects a different feature order or normalization, the deployment should reject incompatible inputs rather than silently produce scores.

## 2. Choose a serving pattern

| Pattern | Useful when | Main tradeoff |
| --- | --- | --- |
| Batch | predictions can be precomputed on a schedule | freshness is limited by the batch interval |
| Online synchronous | a user request needs a response immediately | latency and availability are on the critical path |
| Asynchronous job | work is expensive and completion can be delayed | requires job state, retries, and completion notification |
| Edge/on-device | connectivity or privacy requires local inference | constrained memory, compute, and update process |

Choose based on freshness, latency, throughput, availability, privacy, and cost—not on which framework is most familiar.

## 3. Make a latency budget

For a synchronous request:

$$
T_{\mathrm{total}}=
T_{\mathrm{queue}}+
T_{\mathrm{network}}+
T_{\mathrm{validation}}+
T_{\mathrm{preprocess}}+
T_{\mathrm{model}}+
T_{\mathrm{postprocess}}.
$$

Measure the end-to-end distribution and component distributions. Report p50, p95, and p99; the mean can hide tail failures. Set a deadline and propagate it so that a timed-out request does not keep consuming resources indefinitely.

Batching can improve accelerator utilization while adding queue delay. Measure latency under realistic concurrent load, not just a single request in a notebook.

## 4. Monitor three layers

### Service health

- request volume, error rate, timeout rate;
- p50/p95/p99 latency;
- CPU/GPU, memory, queue depth, and saturation;
- cost per request or successful task.

### Data health

- schema and feature availability;
- missingness, ranges, and category frequencies;
- input volume and freshness;
- group and time composition;
- training-serving feature parity.

### Model and task quality

- delayed-label metrics;
- calibration and action rate when probabilities drive decisions;
- error slices by important population or operating condition;
- drift indicators and human override rate.

Input drift alone does not prove model quality has fallen. A quality alert needs labels or a validated proxy; keep the distinction explicit.

## 5. Roll out in controlled stages

1. Validate the artifact and input contract offline.
2. Run a shadow deployment that logs predictions without changing user outcomes.
3. Compare latency, errors, and input behavior with the current version.
4. Route a small, monitored canary share if the product allows it.
5. Increase traffic only when predeclared health and quality gates pass.
6. Preserve the prior version and a safe fallback.

For models that affect decisions, define rollback criteria before launch. Examples include a sustained error-rate increase, latency above the service objective, a broken feature contract, or a material quality regression on mature labels.

## 6. Incident checklist

When predictions look wrong:

1. determine whether the failure is input, preprocessing, model, serving, or product-policy related;
2. check recent release and data-contract changes;
3. sample permitted, redacted traces;
4. compare feature distributions and model versions;
5. stop or route traffic to a safe baseline if the risk threshold is met;
6. preserve evidence for root-cause analysis;
7. record correction, owner, and prevention test.

Do not retrain automatically just because one drift detector fired. Validate the cause and assess labels, business impact, and safety first.

## 7. Cost per successful task

Raw cost per request can favor a cheap system that fails often. For an evaluation window, compute:

$$
\text{cost per successful task}
=\frac{\text{total serving and dependency cost}}
{\text{number of tasks meeting the success contract}}.
$$

Report the denominator, failure definition, traffic mix, and time window. For generative systems, include retries, tool calls, retrieval, and evaluation costs where material.

## Practice

1. A batch job runs hourly but the product needs a prediction within 100 ms. Which serving pattern is more appropriate, and what would you measure?
2. p50 latency is 30 ms and p99 is 900 ms against a 200 ms deadline. Is the service healthy?
3. Feature distributions shift but no labels are available yet. What can you conclude?
4. A new model has higher offline accuracy but breaks 2% of requests due to a schema mismatch. Should rollout continue?
5. Why preserve a previous artifact and preprocessing version together?

## Answer checks

1. Online synchronous serving is a candidate; measure realistic end-to-end tail latency, throughput, saturation, failure rate, and cost.
2. The tail violates the deadline. The average or median does not satisfy the request-level service expectation.
3. You can report input drift and investigate it. Without labels or a validated proxy, you cannot claim that task quality fell or improved.
4. No. A schema-contract failure is a release blocker; fix the artifact/interface mismatch and rerun validation.
5. Rollback must restore the complete prediction behavior, not weights that depend on a different transformation.

## Mini-project

Take the capstone model and write a runbook that specifies serving mode, request contract, latency budget, monitoring dashboard, release gates, incident owner, fallback, rollback trigger, and a redacted example trace. Simulate an input-schema change and show that the system rejects or safely handles it.

## Common mistakes

- Monitoring CPU and latency while ignoring model quality.
- Calling input drift a performance regression without labels.
- Benchmarking one request instead of realistic concurrency.
- Rolling back only the model weights and forgetting preprocessing.
- Setting alerts without naming an owner or response action.
