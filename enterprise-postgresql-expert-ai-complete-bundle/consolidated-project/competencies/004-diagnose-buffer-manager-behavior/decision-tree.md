# Decision Tree

```text
High query latency
|
+-- BUFFERS available?
|   +-- No --> collect EXPLAIN (ANALYZE, BUFFERS) safely
|
+-- temp read/written present?
|   +-- Yes --> identify spilling sort/hash node
|
+-- shared read high?
|   +-- Yes --> inspect I/O timings and OS cache/storage metrics
|
+-- shared hit high, read low?
|   +-- Yes --> inspect CPU, rows processed, loops, filters
|
+-- rows removed high?
    +-- Yes --> investigate selectivity, indexes, casts, statistics
```
