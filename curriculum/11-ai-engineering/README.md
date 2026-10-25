# AI Engineering

This module brings software engineering, MLOps, deployment, and AI systems design into one place. It teaches how to make AI work reproducible, testable, deployable, observable, and maintainable.

## Outcomes

- Turn notebooks and experiments into maintainable Python projects and APIs.
- Version data, code, models, and configurations so results can be reproduced.
- Test data and model behavior and manage model release and rollback.
- Choose serving patterns and reason about latency, throughput, reliability, and cost.
- Secure AI applications and monitor both service health and task quality.

## Prerequisites

Python, Git, data processing, and a trained ML or DL model. HTTP/API fundamentals and basic command-line use are useful.

## Suggested sequence

### Software foundations

1. [Production Python and project structure](01-production-python-and-project-structure.md)
2. [Git, testing, APIs, and contracts](02-git-testing-apis-and-contracts.md)

### Model lifecycle and deployment

3. [Reproducible deployment and monitoring](03-reproducible-deployment-and-monitoring.md)
4. [Model lifecycle, testing, and monitoring](04-model-lifecycle-testing-and-monitoring.md)
5. [Model serving and inference patterns](05-model-serving-and-inference-patterns.md)
6. [Containers, cloud, and deployment](06-containers-cloud-and-deployment.md)

### Operating production systems

7. [Reliability, observability, and cost](07-reliability-observability-and-cost.md)
8. [Security for AI applications](08-security-for-ai-applications.md)
9. [Inference serving, latency, and cost](09-inference-serving-latency-and-cost.md)
10. [Production model runbook](10-production-model-runbook.md)
11. [LLMOps evaluation and experiment design](11-llmops-evaluation-and-experiment-design.md)

## Practice

- Refactor a notebook into a small Python package with configuration, logs, and a repeatable command.
- Reproduce a saved model result from a clean environment and document data, dependency, and model versions.
- Package one model behind a validated request/response contract and compare batch with online inference.
- Simulate a dependency outage or input schema change; verify timeouts, fallback, monitoring, and rollback.
- Benchmark realistic traffic and report p50/p95/p99 latency, throughput, memory, and cost per successful task.
- Write and rehearse the [production model runbook](10-production-model-runbook.md), including a simulated schema failure and rollback.
- Complete the [LLMOps evaluation project](11-llmops-evaluation-and-experiment-design.md) with a versioned test set, paired comparison, slice analysis, and release gates.

## Checkpoint project

Deliver one end-to-end model application with source and data lineage, input validation, evaluation evidence, deployment configuration, service and task monitoring, security notes, cost estimates, and a tested rollback plan.
