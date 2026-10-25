# SQL and Dataset Construction for AI

SQL is central to building training datasets from operational systems. Correct joins, time filters, and label windows matter as much as the model algorithm.

## 1. Relational thinking

A table has rows and columns; a key identifies an entity or event. A fact table records events; dimension tables add descriptive attributes. Before joining, check key uniqueness and expected cardinality. A one-to-many join can multiply rows and accidentally overweight some examples.

## 2. Core query patterns

```sql
SELECT
    customer_id,
    COUNT(*) AS order_count,
    SUM(amount) AS total_amount,
    MAX(event_time) AS last_order_time
FROM orders
WHERE event_time < :prediction_time
GROUP BY customer_id;
```

Filter to information available by the prediction timestamp. For a training example at time $t$, each feature must be computable from events with event time at or before $t$ and available to the system by the defined cutoff.

Use `LEFT JOIN` when missing related records should remain as examples, and `INNER JOIN` only when excluding unmatched rows is intended. Check row counts before and after each join.

## 3. Window functions

Window functions calculate per-row aggregates without collapsing groups. They are useful for prior-event counts, ranks, and rolling statistics. Use a frame that excludes the current or future event when the prediction must precede that event.

```sql
SELECT
    customer_id,
    event_time,
    SUM(amount) OVER (
        PARTITION BY customer_id
        ORDER BY event_time
        ROWS BETWEEN UNBOUNDED PRECEDING AND 1 PRECEDING
    ) AS amount_before_event
FROM orders;
```

Boundary behavior matters: the first event has no prior events and should be handled intentionally.

## 4. Labels and observation windows

Define a feature window, prediction time, outcome window, and label maturity delay. For example, predict whether a subscription cancels within 30 days using activity from the preceding 90 days. Recent examples may not yet have a full 30-day outcome window; they are censored or immature, not negative.

Avoid selecting rows based on future availability in a way that biases the population. Record exclusions and label versions.

## 5. Efficient and safe extraction

Select only needed columns; filter and aggregate near the data source when appropriate. Partition large datasets by time or stable identifiers. Parameterize queries instead of interpolating user-provided strings. Use transactions or snapshot-consistent reads when multiple tables must represent the same point in time.

## 6. Validate the constructed dataset

Check row counts, unique sample keys, label prevalence, time range, missingness, target windows, join coverage, and leakage-prone columns. Sample records and manually trace them back to source events. Store query/code version, source snapshot, and split assignment with the dataset artifact.

## 7. Exercises

1. A customer has 10 transactions and another has 1. A join produces 10 rows for the first customer. What happens if the target is customer-level?
2. A 30-day churn label is generated for events from yesterday. Why is “not churned” an unsafe label?
3. Why should a windowed feature exclude events after the prediction timestamp?
4. What checks would expose a join that unexpectedly doubled the dataset row count?

## References

- [PostgreSQL documentation: Queries](https://www.postgresql.org/docs/current/queries.html)
- [BigQuery: Window function calls](https://cloud.google.com/bigquery/docs/reference/standard-sql/window-function-calls)
