# Lab

## Inspect WAL Statistics

```sql
SELECT *
FROM pg_stat_wal;
```

## Generate WAL

```sql
CREATE TABLE wal_lab (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    payload text
);

INSERT INTO wal_lab(payload)
SELECT repeat(md5(i::text), 10)
FROM generate_series(1, 100000) AS g(i);
```

Re-check WAL statistics.

## Inspect LSN Movement

```sql
SELECT pg_current_wal_lsn();
```

## Inspect Slots

```sql
SELECT *
FROM pg_replication_slots;
```

## Task

Write a report that distinguishes:

- WAL generation
- WAL retention
- WAL archiving
- WAL replay
