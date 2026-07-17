# Lab

## Inspect Processes

```bash
ps -ef | grep postgres
```

Identify:

- postmaster
- backend processes
- checkpointer
- background writer
- WAL writer
- autovacuum launcher

## Inspect Current Backend

```sql
SELECT pg_backend_pid();
```

Match the PID to the operating-system process.

## Observe Query Lifecycle

```sql
CREATE TABLE architecture_lab (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    payload text NOT NULL
);

INSERT INTO architecture_lab(payload)
SELECT md5(i::text)
FROM generate_series(1, 10000) AS g(i);

EXPLAIN (ANALYZE, BUFFERS)
SELECT *
FROM architecture_lab
WHERE id = 5000;
```

## Questions

- Which process executed the query?
- Which node accessed data?
- Were pages hit or read?
- What cannot be concluded from `shared read` alone?
