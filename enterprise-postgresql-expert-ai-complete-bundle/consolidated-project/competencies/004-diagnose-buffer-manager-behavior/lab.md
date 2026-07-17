# Lab

## Prepare Data

```sql
CREATE TABLE buffer_lab (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    group_id integer NOT NULL,
    payload text NOT NULL
);

INSERT INTO buffer_lab(group_id, payload)
SELECT i % 1000, md5(i::text)
FROM generate_series(1, 1000000) AS g(i);

ANALYZE buffer_lab;
```

## Sequential Scan

```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT *
FROM buffer_lab
WHERE group_id = 42;
```

Record:

- shared hit
- shared read
- rows removed
- execution time

## Add Index

```sql
CREATE INDEX buffer_lab_group_id_idx
ON buffer_lab(group_id);

ANALYZE buffer_lab;
```

Run the same plan again.

## Compare

- plan node
- buffers
- rows removed
- execution time

## Sort Spill

```sql
SET LOCAL work_mem = '1MB';

EXPLAIN (ANALYZE, BUFFERS)
SELECT *
FROM buffer_lab
ORDER BY payload;
```

Observe temporary I/O.

## Safety Note

Use `SET LOCAL` inside a transaction for experiments.
