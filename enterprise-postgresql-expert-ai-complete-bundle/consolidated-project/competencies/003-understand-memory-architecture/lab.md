# Lab

## Inspect Settings

```sql
SHOW shared_buffers;
SHOW work_mem;
SHOW maintenance_work_mem;
SHOW wal_buffers;
SHOW effective_cache_size;
```

## Create a Sort Workload

```sql
CREATE TABLE memory_lab AS
SELECT i AS id, md5(i::text) AS payload
FROM generate_series(1, 500000) AS g(i);

ANALYZE memory_lab;

EXPLAIN (ANALYZE, BUFFERS)
SELECT *
FROM memory_lab
ORDER BY payload;
```

Observe:

- Sort Method
- Memory
- Disk
- temp read
- temp written

## Query-Local Test

```sql
BEGIN;
SET LOCAL work_mem = '128MB';

EXPLAIN (ANALYZE, BUFFERS)
SELECT *
FROM memory_lab
ORDER BY payload;

ROLLBACK;
```

Compare before and after.

Do not convert the test into a global recommendation without concurrency analysis.
