# Lab

## Create Table

```sql
CREATE TABLE mvcc_lab (
    id integer PRIMARY KEY,
    payload text
);

INSERT INTO mvcc_lab VALUES (1, 'old');
```

## Session A

```sql
BEGIN;
SELECT * FROM mvcc_lab WHERE id = 1;
```

Keep transaction open.

## Session B

```sql
UPDATE mvcc_lab
SET payload = 'new'
WHERE id = 1;

COMMIT;
```

## Session A

```sql
SELECT * FROM mvcc_lab WHERE id = 1;
```

Observe visibility according to isolation level and transaction boundaries.

## Inspect Activity

```sql
SELECT pid, xact_start, state, query
FROM pg_stat_activity
WHERE xact_start IS NOT NULL;
```

## Task

Explain why old tuple versions may remain necessary.
