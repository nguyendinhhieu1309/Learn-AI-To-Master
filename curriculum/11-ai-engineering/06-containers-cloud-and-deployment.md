# Containers, Cloud, and Deployment Architecture

## Learning objectives

Understand how to package an AI application, identify the cloud resources it needs, and plan a release that can be observed and rolled back.

## 1. Containers

A container image packages application code, runtime, and dependencies. It helps keep development and deployment environments aligned. It does not automatically contain model weights, credentials, GPU drivers, or external services; define how those are mounted or fetched and validate compatibility.

Keep images reproducible and reasonably small: use explicit base images, install pinned dependencies, avoid copying local secrets/data, and run as a non-root user when feasible. Separate build-time configuration from runtime secrets.

## 2. Cloud building blocks

An AI system commonly uses compute (CPU/GPU), object storage for datasets/artifacts, a database or cache for application state, a queue for asynchronous jobs, a model endpoint, and observability services. Choose managed or self-hosted components based on operational skill, scale, latency, data residency, cost, and lock-in requirements.

Use identity and access management to grant each service the minimum permissions it needs. A model-serving container should not have broad access to unrelated storage or databases.

## 3. CI/CD and environments

Continuous integration can run formatting, static checks, unit/contract tests, security scanning, and selected model evaluations on proposed changes. A deployment pipeline promotes the same build artifact through development, staging, and production with environment-specific configuration. Model/data artifacts should have their own versions and approval criteria.

Use canary or shadow release where risk and architecture permit. Record the release version and provide rollback. Infrastructure-as-code can make cloud resources reviewable and reproducible.

## 4. Kubernetes and orchestration (conceptual)

Container orchestration schedules workloads, restarts failed processes, and can scale replicas. Kubernetes provides primitives such as pods, deployments, services, and jobs. It adds operational complexity: cluster security, networking, GPU scheduling, observability, upgrades, and capacity. Learn the concepts, but do not require Kubernetes for a small single-service project.

## 5. Accelerator and model artifacts

Match runtime libraries, device drivers, model format, and GPU capability. Large model weights may be downloaded at startup or mounted from artifact storage; consider cold-start time, checksum/verification, license, and cache policy. Quantized models can reduce memory and cost but must be reevaluated for quality.

## Practice

Draw an architecture for a private document assistant. Mark where source files, vector index, model endpoint, API credentials, conversation state, and traces live. For each component, define its access policy, backup/recovery expectation, and version information.

## Common mistakes

- Putting credentials into an image layer or container environment visible to logs.
- Treating a container as proof that deployment is reproducible.
- Introducing orchestration before understanding the workload and team operations.
- Updating model files in place without checksums, versions, or rollback.
