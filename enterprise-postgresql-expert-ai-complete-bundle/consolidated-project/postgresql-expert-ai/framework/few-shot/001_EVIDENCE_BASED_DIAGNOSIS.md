---
id: few-shot-001
title: Evidence-Based Diagnosis Examples
category: Few Shot
language: English
version: 1.0.0
---

# Few-Shot Examples: Evidence-Based Diagnosis

## Example 1: Sequential Scan with high filter rejection

### User input

```text
Seq Scan on orders
  (cost=0.00..180000.00 rows=100 width=96)
  (actual time=0.040..850.000 rows=120 loops=1)
  Filter: (customer_id = 42)
  Rows Removed by Filter: 4999880
  Buffers: shared hit=12000 read=62000
  I/O Timings: shared read=610.000
Execution Time: 855.000 ms
```

### Expected answer

#### Summary

The query scans roughly five million rows to return 120 rows. The cardinality estimate is accurate, but the access path is inefficient for this predicate. The main cost is the large table scan and associated page reads.

#### Evidence

- Actual rows returned: 120
- Rows removed by filter: 4,999,880
- Total rows examined: approximately 5,000,000
- `shared read=62000`
- Read timing: 610 ms

#### How it works

A Sequential Scan evaluates the predicate against each visible tuple in the table. The scan can still be the correct plan when many rows are required, but here only 120 rows survive the filter.

#### Calculation

```text
62,000 pages × 8 KiB = 496,000 KiB ≈ 484 MiB
```

#### Diagnosis

The query uses a non-selective access path for a highly selective predicate.

#### Recommended action

First verify that no suitable index exists and that the predicate does not apply a function or cast to the indexed column.

```sql
CREATE INDEX CONCURRENTLY idx_orders_customer_id
ON orders (customer_id);

ANALYZE orders;
```

Then rerun:

```sql
EXPLAIN (ANALYZE, BUFFERS, SETTINGS)
SELECT *
FROM orders
WHERE customer_id = 42;
```

#### Validation

Compare execution time, `shared read`, rows removed by filter, and the selected access node.

#### Key lesson

Do not recommend an index merely because a Sequential Scan appears. Recommend it because the predicate is selective and the plan performs excessive filtering.

## Example 2: Nested Loop multiplication

### User input

```text
Nested Loop
  (actual time=0.100..4200.000 rows=200000 loops=1)
  -> Seq Scan on orders
       (actual rows=200000 loops=1)
  -> Index Scan using customers_pkey on customers
       (actual time=0.005..0.010 rows=1 loops=200000)
```

### Expected answer

The inner Index Scan is individually fast but executes 200,000 times. The approximate upper-bound multiplication is:

```text
0.010 ms × 200,000 ≈ 2,000 ms
```

This does not prove that the Index Scan alone consumed exactly 2,000 ms, because node timings are inclusive and instrumentation has overhead. It does show why high `loops` values must be evaluated as multiplicative work.

Check estimate accuracy, outer-row reduction, join selectivity, and whether `Hash Join` would be more appropriate. Do not force-disable Nested Loop globally.

## Example 3: Sort spill

### User input

```text
Sort
  (actual time=1200.000..1600.000 rows=3000000 loops=1)
  Sort Method: external merge  Disk: 350000kB
  Buffers: shared hit=20000 read=40000
  temp read=90000 written=91000
```

### Expected answer

The sort exceeded its memory budget and used an external merge operation. The evidence is `external merge`, disk usage, and temporary reads and writes.

```text
90,000 temp pages × 8 KiB ≈ 703 MiB read
91,000 temp pages × 8 KiB ≈ 711 MiB written
```

Test a session-local `work_mem` value, but also inspect whether an index can satisfy the `ORDER BY`, whether `LIMIT` enables a top-N strategy, and whether rows can be filtered before sorting.

```sql
BEGIN;
SET LOCAL work_mem = '256MB';
EXPLAIN (ANALYZE, BUFFERS, SETTINGS)
SELECT ...;
ROLLBACK;
```

Never raise global `work_mem` based on one query without considering concurrent sort and hash nodes.
