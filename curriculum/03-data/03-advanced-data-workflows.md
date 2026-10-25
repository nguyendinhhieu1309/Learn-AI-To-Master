# Advanced Data Workflows

## Learning objectives

Choose storage and processing patterns for a dataset, query relational data safely, handle event/time-series data, and keep data lineage and quality visible as a workflow grows.

## 1. SQL and relational data

A relational database stores records in tables. A **primary key** identifies a row; a **foreign key** links it to another table. Before joining tables, ask what one row represents on each side. If both sides have repeated keys, a join can multiply rows and inflate totals.

```sql
SELECT c.region, COUNT(DISTINCT o.order_id) AS orders,
       SUM(o.amount) AS revenue
FROM orders AS o
JOIN customers AS c ON o.customer_id = c.customer_id
WHERE o.created_at >= '2025-01-01'
GROUP BY c.region
ORDER BY revenue DESC;
```

`WHERE` filters rows before aggregation; `GROUP BY` defines groups; `HAVING` filters groups after aggregation. `NULL` means unknown/absent, not zero. Comparisons such as `column = NULL` do not work as expected; use `IS NULL`. Check row counts and totals before and after joins.

## 2. Choose a storage format

- **CSV:** easy to inspect and exchange; weak typing and inefficient for large repeated scans.
- **JSON/JSONL:** flexible records and nested structure; often larger and slower to scan column-wise.
- **Parquet:** typed, compressed, columnar; useful for analytical workloads that read selected columns.
- **Database tables:** useful for shared structured data and controlled queries.
- **Media files:** store images/audio/video separately and keep paths, metadata, labels, and licenses in an index.

Retain a documented raw source and write processed outputs separately. Avoid silently overwriting source files.

## 3. Process data larger than memory

First measure size and identify whether the workload is I/O-, CPU-, or memory-bound. Select columns and filter early, use efficient dtypes, vectorize operations, and process in chunks when possible. Partition data by a useful key/date, but avoid so many tiny files that metadata overhead dominates.

Distributed computing adds network, scheduling, serialization, and consistency costs. Use it when a single-machine approach is a real bottleneck, not as a default. Before scaling out, check whether a simpler columnar format, chunked read, or SQL aggregation solves the problem.

## 4. Time series and streaming data

For forecasting, use event time (when the measurement happened) rather than only processing time (when the system received it). Sort and validate timestamps, handle time zones and duplicates, and define windows explicitly. A rolling feature for time `t` may use only observations available before the prediction cutoff; including future values is leakage.

In event streams, data may arrive late, out of order, or more than once. Define deduplication keys, late-event policy, window duration, and whether results can be revised. Monitor gaps, clock drift, and backpressure.

## 5. Specialized data needs

- **Text:** preserve language, document source, timestamps, and access permissions; detect encoding issues and duplicate/near-duplicate documents.
- **Images/video:** preserve camera, resolution, timestamp, scene/person identity, label version, and train/validation/test assignment. Split by source scene when nearby frames are correlated.
- **Sensors/robotics:** preserve units, frame IDs, calibration version, and synchronized timestamps. Do not combine measurements from different frames without a transform.
- **Labels:** track who/what generated them, annotation guidelines, confidence, and corrections.

## 6. Lineage, privacy, and quality over time

Record data source/version, collection time, license, transformations, schema, and output version. A lightweight data card should say how data were collected, what is missing, known biases, permitted use, and intended limitations. Minimize personal data; restrict access and retention; remove secrets from logs and samples.

Data quality checks should run each time the pipeline runs: required columns, type/range constraints, duplicate rates, missingness, allowed categories, time coverage, and label balance. Compare changes with a known reference and investigate material shifts. Do not automatically “repair” a shift without checking the cause.

## Practice

Design an ingestion pipeline for daily orders. Write its row meaning, key, schema, duplicate rule, missing-value policy, timezone, and validation checks. Explain how you would detect a join that accidentally doubles revenue. For a future-demand model, list all candidate features that might contain information from after the forecast cutoff.

## Common mistakes

- Joining tables without checking key uniqueness and row counts.
- Converting `NULL` to zero without knowing whether zero is meaningful.
- Scaling to a distributed platform before measuring the bottleneck.
- Treating a data-quality alert as permission to drop or rewrite records automatically.
- Losing provenance when creating a cleaned dataset or annotation export.
