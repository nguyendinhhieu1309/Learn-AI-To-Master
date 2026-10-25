# SQL, Databases, and Analytical Data Models

## Learning objectives

Query related data without accidentally multiplying rows, understand transactional vs. analytical workloads, and model data for reliable reporting and ML.

## 1. Relational tables and keys

A table has rows and columns. A **primary key** uniquely identifies a row. A **foreign key** references a row in another table. Before joining, learn the grain of each table: one row per customer, order, order line, or event? Grain determines which aggregations are valid.

Example tables: `customers(customer_id, region)` and `orders(order_id, customer_id, amount, created_at)`. `customer_id` may repeat in orders because one customer can place many orders.

## 2. Join cardinality

An inner join keeps matching keys; a left join keeps every left row and adds matching right-side values. If both tables contain multiple rows per join key, one row can match many and multiply results. Check uniqueness and counts before aggregation.

```sql
SELECT c.region, COUNT(DISTINCT o.order_id) AS order_count,
       SUM(o.amount) AS revenue
FROM customers AS c
JOIN orders AS o USING (customer_id)
WHERE o.created_at >= DATE '2025-01-01'
GROUP BY c.region
ORDER BY revenue DESC;
```

For order-line data, sum line amounts at the line grain or aggregate lines to one order row before joining another one-to-many table.

## 3. SQL essentials

- `SELECT` chooses expressions/columns; `WHERE` filters source rows.
- `GROUP BY` creates aggregation groups; `HAVING` filters groups after aggregation.
- `ORDER BY` orders results; `LIMIT` caps returned rows in many systems.
- `NULL` means unknown or missing; test with `IS NULL`, not `= NULL`.
- Use explicit column names and aliases; avoid `SELECT *` in durable pipeline contracts.

Use parameterized queries for user-provided values. Never build SQL by concatenating raw user text.

## 4. Transactional vs. analytical systems

OLTP systems handle many small reads/writes for application transactions. OLAP systems optimize scans and aggregations over large histories. A data warehouse often organizes cleaned, integrated records for analysis. A star schema has a central fact table (events/measures) and dimension tables (who, what, where, when). This makes the grain visible and helps analytics avoid contradictory joins.

Indexes can speed lookups or joins but take storage and slow writes. Columnar formats such as Parquet scan selected columns efficiently; row-oriented tables can suit transactional access. Choose based on queries, update patterns, and operational needs.

## 5. Data modeling for ML

Create an explicit prediction table with one row per decision point, a timestamp/cutoff, features available at that time, and a target defined over a future window. Keep entity IDs and source timestamps long enough to validate grouping and leakage, even if they are not model features. Document how labels are generated and delayed.

## Practice

Take two tables with repeated keys. Predict the number of rows after an inner join before running it. Check key counts, run the join, compare row counts and total amount before/after, and correct the query if amounts are duplicated.

## Common mistakes

- Joining on a non-unique key and multiplying facts.
- Treating NULL, zero, and “not applicable” as the same value.
- Including post-outcome data in a feature table.
- Building an analytics table without documenting its grain.
