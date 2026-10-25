# ETL/ELT Pipelines and Orchestration

## Learning objectives

Design data jobs that can be rerun safely, make dependencies explicit, and detect quality problems before a model is trained on bad inputs.

## 1. ETL and ELT

**ETL** extracts source data, transforms it, then loads curated results into storage. **ELT** loads raw or lightly processed data first, then transforms within the destination platform. Both need provenance, access control, validation, and recovery. The right choice depends on data volume, latency, privacy, compute, and platform.

## 2. A pipeline as a directed acyclic graph

A scheduled workflow may contain: ingest → schema validation → deduplication → join → feature creation → quality checks → publish. A directed acyclic graph (DAG) makes dependencies visible and allows a scheduler to retry or run independent steps concurrently.

Keep each task small and observable. Store large intermediate data in object/database storage; pass references and metadata between tasks rather than loading everything into a scheduler's memory.

## 3. Idempotency and incremental work

A job is **idempotent** if running it twice produces the same intended state as running it once. Use stable keys, deterministic transformations, atomic writes, and partition replacement/merge strategies. Record watermarks for incremental ingestion. A retry should not duplicate rows or repeat external side effects.

Distinguish at-least-once delivery (events may repeat), at-most-once (events may be lost), and exactly-once claims, which usually require coordinated storage/processing semantics. Design deduplication and reconciliation even when a platform offers strong guarantees.

## 4. Quality gates and schema evolution

Check required columns/types, uniqueness, ranges, null rates, category values, time coverage, freshness, and row-count changes. Define which conditions fail the job and which create warnings for investigation. Schema changes should be versioned and reviewed; adding an optional field differs from renaming a required field.

Do not publish a partially written dataset. Write to a temporary/versioned location, validate, then atomically mark the new version current when the storage system supports it.

## 5. Lineage, observability, and recovery

Record source versions, code revision, run ID, input/output partitions, counts, duration, and validation results. On failure, preserve logs and partial metadata, alert an owner, and define whether to retry, backfill, or roll back. A pipeline that silently skips a bad day creates missing training data that may be hard to notice later.

## 6. Feature production

Separate reusable feature definitions from ad hoc notebook code. Validate that training and serving transformations agree. For a feature with delayed data, define a point-in-time join so each training row sees only values available at its prediction timestamp.

## Practice

Design a daily pipeline that ingests orders. Specify its idempotency key, watermarks, late-arriving event policy, quality gates, output versioning, alert owner, and backfill procedure. Describe how a job retry avoids duplicating orders.

## Common mistakes

- Treating a scheduler retry as harmless when steps are not idempotent.
- Writing a partial dataset into the location consumers already read.
- Monitoring job success but not freshness or data volume.
- Rebuilding historical features with current values and creating time leakage.
