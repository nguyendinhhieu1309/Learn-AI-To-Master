# Batch, Streaming, and Feature Consistency

## Learning objectives

Choose batch or streaming based on product needs, reason about late and duplicate events, and keep features consistent between model training and online predictions.

## 1. Batch vs. streaming

Batch processing handles a bounded dataset on a schedule: a nightly report or daily model refresh. Streaming processes events continuously or in small windows. Streaming can reduce latency but adds complexity: ordering, duplicates, backpressure, checkpointing, late data, and replay.

Choose the simplest approach that meets freshness needs. A near-real-time requirement should be stated in measurable terms (for example, p95 data freshness under a defined threshold) before designing a streaming platform.

## 2. Event time and processing time

**Event time** is when something happened; **processing time** is when the system received it. Events can arrive late or out of order. Windowed aggregates need a policy for late arrivals and revisions. Forecasting features must use only events received by the decision cutoff, even if an event claims an earlier timestamp but arrived later.

## 3. Training/serving skew

Training/serving skew occurs when a feature is computed differently in historical data and live inference. Examples: a time zone mismatch, different category mappings, an unavailable source field, or an aggregate computed over different windows. Centralize transformation logic where possible and compare feature values from batch and online paths on the same examples.

## 4. Feature stores (conceptual model)

A feature store manages feature definitions and values for reuse. An **offline store** serves historical training/analysis; an **online store** serves low-latency inference. The two must represent compatible definitions and timestamps. Point-in-time correctness matters: historical examples should only use feature values that were available at their prediction time.

A feature store is a tool, not a cure for poor semantics. Define entity key, feature timestamp, freshness, null behavior, ownership, and access policy first.

## 5. Data versioning and snapshots

Version raw inputs or immutable snapshots so an experiment can be recreated. Record schema, source, code, labels, and transformation versions. When data are too large to copy, use content hashes, partition manifests, or snapshot-capable storage and document retention.

## Practice

For a fraud model, define one rolling spending feature. Specify its event-time window, late-event treatment, whether a transaction itself is included, how training reconstructs the feature as of each historical timestamp, and how online inference reads a fresh value.

## Common mistakes

- Choosing streaming because it sounds more modern when hourly batch would suffice.
- Ignoring event arrival time and using future information in replayed history.
- Computing offline features with a different null/default rule than online features.
- Assuming an online store is current without monitoring freshness and failures.
