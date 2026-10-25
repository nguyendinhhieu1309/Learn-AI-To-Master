# Reproducible Deployment and Monitoring

## Learning objectives

Preserve the relationship between data, preprocessing, model, and output; serve predictions consistently; and monitor the system after release.

## 1. Reproduce a model result

Record source data/version, split definition, code revision, dependencies, configuration, random seed, hardware where relevant, and metrics. Save the fitted preprocessing along with the estimator. A serialized model without its tokenizer, scaler, category mapping, or label order is incomplete.

Reproduction means another person can follow documented steps and get a comparable result. Floating-point and accelerator differences can prevent bit-for-bit identity; explain tolerances and variance.

## 2. Package inference

Define a stable input/output schema: required fields, types, units, allowed ranges, and behavior for missing values. Apply exactly the same preprocessing as training. Validate requests, set size/time limits, and return explicit errors rather than silently changing malformed inputs.

For batch jobs, throughput may dominate. For interactive APIs or robotics, tail latency and availability may matter more. Measure on the target runtime and hardware.

## 3. Release safely

Use separate development, staging, and production configurations. Check known examples, invalid inputs, and model/preprocessor compatibility before release. Version artifacts and retain a rollback path. A shadow or canary deployment can reveal real input differences before wide exposure, subject to privacy and operational constraints.

## 4. Monitor service and model

Service monitoring includes errors, latency percentiles, throughput, resource use, and availability. Data monitoring includes schema changes, missing rates, feature ranges, category frequencies, and input drift. Model monitoring includes delayed labels, quality metrics, calibration, and group-level behavior where appropriate.

Input drift is a signal to investigate, not proof that performance fell; stable input distributions do not prove the model is still accurate. Use labels or a suitable audit to measure quality. Define who responds and what actions an alert triggers.

## 5. Documentation and governance

Keep a model card describing intended use, evaluation data, metrics, subgroup results where available, limitations, and maintenance owner. Respect data licenses, privacy, retention, access control, and applicable organizational requirements. For robotics or other high-impact systems, document safe states and independent safety controls.

## Practice

Package one model with preprocessing into a small inference function or service. Test valid, missing, out-of-range, and unknown-category inputs. Write down the artifact version and a rollback plan. Identify one quality metric that can be monitored only after labels arrive.

## Common mistakes

- Updating preprocessing separately from the model.
- Alerting on drift without an owner or a defined investigation step.
- Automatically retraining on new data without data and label quality checks.
- Treating a high offline score as proof of safe production performance.
