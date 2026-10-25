# Data Processing and Engineering: From Basics to Production Pipelines

Data quality and data handling shape every downstream result. This single module moves from inspecting and cleaning a table to SQL, reliable batch/stream pipelines, feature consistency, and data quality operations.

## Outcomes

- Inspect, clean, transform, and visualize raw data.
- Construct datasets with safe joins, time windows, and point-in-time features.
- Design repeatable batch and streaming pipelines and monitor their quality.

## Prerequisites

Basic Python is recommended. Learn NumPy and pandas as you progress.

## Suggested sequence

1. [Clean and explore a tabular dataset](01-cleaning-and-exploratory-analysis.md)
2. [Splits, pipelines, and data validation](02-splits-pipelines-and-validation.md)
3. [Advanced data workflows](03-advanced-data-workflows.md)
4. [Data cleaning, EDA, and feature engineering](04-data-cleaning-eda-and-feature-engineering.md)
5. [SQL and dataset construction](05-sql-and-dataset-construction.md)
6. [SQL databases and analytical models](06-sql-databases-and-analytical-models.md)
7. [ETL/ELT pipelines and orchestration](07-etl-elt-pipelines-and-orchestration.md)
8. [Batch, streaming, and feature consistency](08-batch-streaming-and-feature-consistency.md)
9. [Data quality, feature stores, and streaming](09-data-quality-feature-stores-and-streaming.md)
10. [Data validation and leakage workshop](10-validation-and-leakage-workshop.md)

### Topic coverage

- **Data model and quality:** schema, row grain, keys, units, labels, missingness, duplicates, invalid values, and outliers.
- **Loading and exploration:** CSV, JSON, Parquet, images, queries, distributions, relationships, imbalance, and sampling bias.
- **Cleaning and features:** joins, grouping, reshaping, encoding, scaling, time features, and domain-informed transformations.
- **Validation without leakage:** stratified, group, and temporal splits; training-only preprocessing; point-in-time joins and label windows.
- **SQL and dataset construction:** analytical joins, window functions, query cardinality, event time, label maturity, and snapshots.
- **Pipelines:** idempotent ETL/ELT jobs, retries, checkpoints, atomic writes, orchestration, and backfills.
- **Batch and streaming:** event time, watermarks, late data, deduplication, freshness, and throughput tradeoffs.
- **Feature operations:** offline/online parity, feature ownership, schema evolution, data contracts, drift signals, and lineage.
- **Specialized data:** text, images/video, time series, point clouds, and sensor data; preserve metadata and alignment.
- **Storage and scale:** choose formats, process chunks, and understand when parallel or distributed processing is warranted.

## Practice

- Audit a dataset schema and report suspicious values before cleaning.
- Compare mean imputation, a missingness indicator, and dropping rows; state when each could mislead.
- Demonstrate target leakage with a deliberately invalid feature, then remove it.
- Build a pipeline that can be run twice and produces the same validated output.
- Complete the [data and split contract workshop](10-validation-and-leakage-workshop.md) before training a model on a time-dependent or grouped dataset.
- Solve the [data engineering problem set](PROBLEM_SET_WITH_SOLUTIONS.md), especially join grain, point-in-time correctness, and idempotent retries.

## Checkpoint project

Choose an openly licensed dataset. Write a short data card, explore and clean the data, create a problem-appropriate split, and build a repeatable preprocessing pipeline. Document excluded records, assumptions, leakage risks, and limitations.

## Common mistakes to watch for

- Cleaning the full dataset before splitting when a transformation learns from data.
- Randomly splitting time-dependent or grouped observations.
- Treating an unusual value as an error without domain context.
- Reporting a polished plot while hiding missingness, class balance, or sampling bias.
