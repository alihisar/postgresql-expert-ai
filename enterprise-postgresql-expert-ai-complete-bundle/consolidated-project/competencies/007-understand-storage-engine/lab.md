# Lab

## Create Wide Rows

```sql
CREATE TABLE storage_lab (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    payload text
);

INSERT INTO storage_lab(payload)
SELECT repeat(md5(i::text), 100)
FROM generate_series(1, 50000) AS g(i);
```

## Inspect Relation Size

```sql
SELECT
    pg_size_pretty(pg_relation_size('storage_lab')) AS heap,
    pg_size_pretty(pg_total_relation_size('storage_lab')) AS total;
```

## Create Index

```sql
CREATE INDEX storage_lab_id_idx
ON storage_lab(id);
```

## Observe Index-Only Behavior

```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT id
FROM storage_lab
WHERE id BETWEEN 100 AND 1000;
```

Record Heap Fetches.

Then:

```sql
VACUUM (ANALYZE) storage_lab;
```

Run again and compare.

## Task

Explain how visibility information changes behavior.
