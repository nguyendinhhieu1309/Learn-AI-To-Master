# Model Lifecycle, Testing, and Monitoring

An ML model is a versioned system of data, transformations, code, parameters, and runtime dependencies. Reliable MLOps makes that whole system reproducible, testable, observable, and reversible.

## 1. Version the model inputs and outputs

Track at least:

- Source revision and dependency lockfile.
- Dataset snapshot, schema, labels, split definition, and preprocessing.
- Training configuration, seed, hardware, and metrics.
- Model artifact, tokenizer or feature map, calibration, and decision threshold.
- Evaluation report, known limits, approval status, and deployment version.

A model file without its preprocessing and label mapping is not a deployable model. Use content hashes or immutable dataset versions where possible.

## 2. Test layers of the ML system

- **Unit tests** for deterministic transforms, feature logic, and postprocessing.
- **Data tests** for schema, ranges, nulls, uniqueness, and label validity.
- **Training tests** for a small smoke run, finite loss, gradients, and expected artifact creation.
- **Model tests** for output shape, probability range, invariants, and minimum baseline metrics.
- **Integration tests** for request parsing, serialization, model loading, and response contracts.
- **Performance tests** for latency, throughput, memory, and startup time.
- **Robustness tests** for missing values, malformed requests, and known edge cases.

Tests should catch regressions without encoding the entire training set or silently accepting changed label semantics.

## 3. Model registry and promotion

A registry associates artifacts with metadata, evaluations, and lifecycle state. Promotion gates should enforce requirements relevant to the risk: quality thresholds, slice checks, latency budget, security review, and reproducible lineage. Keep a staging environment representative of production and a rollback path to a known-good version.

Shadow deployment runs a candidate on real inputs without serving its output. Canary deployment exposes the candidate to a small portion of traffic, with guardrails and automatic rollback criteria. These techniques reduce the blast radius but do not replace offline evaluation.

## 4. Production monitoring

Monitor service health (errors, latency, saturation) and model behavior (input schema, feature distributions, prediction distributions, confidence, abstentions, and delayed labels). Input drift is a signal to investigate; it does not prove performance has fallen. Conversely, stable input distributions do not guarantee stable labels or task relationships.

When labels arrive late, use proxy signals carefully. Periodically evaluate newly labeled production examples using the same label policy and compare by time and meaningful segment. Monitor data freshness and missingness because a pipeline failure can resemble model drift.

## 5. Retraining and rollback

Retraining can be triggered by a schedule, new labeled data, a measured quality drop, or a change in upstream data. Define the trigger, data inclusion rules, evaluation gates, approver, deployment plan, and rollback conditions before automating retraining. A new model should not replace production solely because its training job succeeded.

## 6. Responsible data handling

Limit retained data, protect sensitive fields, control access to logs, and define deletion and retention behavior. Do not log raw prompts, images, or identifiers by default when they are not needed. Restrict model artifact and dataset access according to the sensitivity of their contents.

## 7. Exercises

1. A registry contains a model file but no preprocessing code or label mapping. What is missing for safe deployment?
2. Input drift alarms, but delayed labels show stable performance. What should the response be?
3. A candidate model has better offline accuracy but 4x latency. Which promotion checks apply?
4. The new deployment’s error rate rises only for one API version. How can versioned monitoring narrow the cause?

## References

- [MLflow Model Registry](https://mlflow.org/docs/latest/model-registry.html)
- [Google Rules of Machine Learning](https://developers.google.com/machine-learning/guides/rules-of-ml)
- [Evidently: ML monitoring](https://docs.evidentlyai.com/)
