# Data Quality, Feature Stores, and Streaming

Reliable AI starts with reliable data. A pipeline must preserve meaning, time, lineage, and quality as data moves from source systems to training and inference.

## 1. Data contracts and validation

A data contract describes a table or event stream’s schema, semantics, ownership, freshness, and compatibility expectations. Validate more than column names:

- Types, null rates, ranges, uniqueness, and allowed categories.
- Key relationships and duplicate events.
- Timestamp timezone, event time versus processing time, and freshness.
- Label definition, label delay, and known censoring.
- Distribution changes for important groups and critical features.

Fail loudly when a required field disappears or changes meaning. For gradual distribution changes, alert and investigate rather than silently dropping rows. Record which validation rules ran and their outcomes with each dataset build.

## 2. Idempotent and recoverable pipelines

An idempotent pipeline can be rerun for the same inputs without duplicating or corrupting its output. Use stable keys, deterministic transformations where possible, atomic writes, checkpoints, and explicit backfill behavior. Separate raw immutable inputs from curated datasets so transformations can be reproduced after logic changes.

Track lineage from source partitions through transformations to model artifacts. For each training run, be able to answer: which data version, label policy, preprocessing code, and time cutoff produced this model?

## 3. Event time, late data, and streaming

Streaming systems receive events asynchronously. Event time is when an event happened; processing time is when the system observed it. Late and out-of-order events require a policy: wait with a watermark, revise prior aggregates, or drop events beyond a documented lateness window.

Exactly-once end-to-end delivery is difficult across independent systems. Design consumers to be idempotent, deduplicate using event identifiers, and monitor backlog and late-event rates. A real-time feature should specify freshness and behavior when unavailable.

## 4. Feature stores and point-in-time correctness

A feature store can provide consistent feature definitions for offline training and online serving. The key challenge is point-in-time correctness: a training row at time $t$ must use only feature values that were available by $t$.

Joining the latest feature value to a historical training row can leak future information. Use event timestamps and point-in-time joins; validate by sampling rows and tracing the source records. Track feature ownership, definition, units, defaults, freshness, and deprecation.

Offline/online skew can arise from different code paths, timezone handling, null defaults, aggregation windows, or stale caches. Share transformation logic where practical and run parity checks over the same example records.

## 5. Data quality versus distribution drift

Schema violations, missing features, duplicate events, stale data, and label pipeline failures are data-quality problems. A population shift can be valid new data and is a drift signal. These need different responses: fix broken ingestion; investigate valid drift with outcome labels and product context.

Monitor feature distributions by time and meaningful cohorts, but do not treat one generic drift score as a model-quality metric. A feature can drift without harming predictions, or remain stable while the relationship to labels changes.

## 6. Dataset design for machine learning

Choose a sampling frame that reflects intended use. Store stable record IDs and split assignments so experiments compare the same examples. Document exclusions, deduplication, label adjudication, and subgroup coverage. Keep test data access controlled to prevent iterative tuning against it.

When labels are delayed, distinguish unlabeled recent data from negative examples. Censoring occurs when some outcomes have not had enough time to materialize; naive labeling can systematically bias evaluation.

## 7. Exercises

1. A historical training row is joined to the latest customer balance, which may have been updated after the target outcome. What is the risk and the remedy?
2. An event is processed twice after a retry. How should the pipeline prevent duplicate aggregates?
3. The online feature default is zero while offline training fills missing values with the median. Which system failure can this create?
4. Feature drift rises but quality remains stable on labeled traffic. What does the drift alert establish, and what does it not establish?

## References

- [Feast: Point-in-time joins](https://docs.feast.dev/getting-started/concepts/point-in-time-joins)
- [TFX: Data validation](https://www.tensorflow.org/tfx/guide/tfdv)
- [Google Cloud: Dataflow event time and watermarks](https://cloud.google.com/dataflow/docs/concepts/streaming-pipelines)
