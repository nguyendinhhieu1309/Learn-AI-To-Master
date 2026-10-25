# Reliability, Observability, and Cost

## Learning objectives

Use logs, metrics, and traces to understand AI application behavior; define service objectives; and connect alerts to actionable responses.

## 1. Logs, metrics, and traces

- **Logs** record discrete events (validation failure, model loaded, tool timed out).
- **Metrics** aggregate time-series values (requests/sec, p95 latency, error rate, token usage, queue depth).
- **Traces** connect steps within one request (API → retrieval → model → tool → response).

Use correlation IDs to follow a request through services. Redact or avoid storing personal content and secrets. Keep retention and access controls appropriate to the data.

## 2. Service-level indicators and objectives

A service-level indicator (SLI) is a measured property such as availability, error rate, or p95 latency. A service-level objective (SLO) defines a target over a time window. Choose indicators that reflect user experience, not only whether a process is alive. For a model application, include quality or safe-completion signals where labels or audits are available.

## 3. Model/application monitoring

Monitor feature/schema changes, missingness, prediction distributions, confidence/calibration where meaningful, subgroup performance, delayed labels, retrieval hit rates, citation support, tool failure rates, and model/provider versions. Input drift alone is not proof of quality loss; stable inputs do not guarantee quality. Pair signals with audits or labels.

## 4. Alerting and incident response

An alert needs an owner, severity, and action. Define runbooks for provider outage, model-load failure, sudden latency increase, corrupt data, unsafe action, or quality regression. Include rollback, degraded mode, user communication, and incident notes. Avoid noisy alerts that train operators to ignore them.

## 5. Cost model

Estimate total cost: model input/output, embeddings, retrieval, database, GPU/CPU time, storage, network, monitoring, retries, and human review. Track cost per successful task and by workload/tenant where appropriate. Budget controls can cap tokens, steps, concurrency, and retention. Review quality after each optimization; the cheapest output is not useful if it fails the task.

## 6. Reliability patterns

Use bounded retries with exponential backoff and jitter for transient failures, idempotency keys for side effects, circuit breakers for failing dependencies, and backpressure for overload. Avoid retry storms. Make degraded/fallback behavior visible to the user and logs.

## Practice

For an LLM support assistant, define an SLO for response latency and availability, two application-quality indicators, a provider-outage fallback, a cost-per-success dashboard, and one high-severity alert runbook.

## Common mistakes

- Monitoring only CPU and server uptime while ignoring task failures.
- Alerting on a statistic without defining a response owner.
- Logging entire prompts and retrieved documents by default.
- Using automatic retries that amplify overload or duplicate side effects.
