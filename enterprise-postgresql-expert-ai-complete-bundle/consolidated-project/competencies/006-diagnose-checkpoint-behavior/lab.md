# Lab

## Enable Checkpoint Logging

Use an appropriate test environment.

```conf
log_checkpoints = on
```

## Generate Write Load

```sql
CREATE TABLE checkpoint_lab (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    payload text
);

INSERT INTO checkpoint_lab(payload)
SELECT repeat(md5(i::text), 5)
FROM generate_series(1, 500000) AS g(i);
```

## Force a Checkpoint

```sql
CHECKPOINT;
```

## Observe

- checkpoint duration
- WAL growth
- disk latency
- write throughput
- application latency

## Safety

Do not repeatedly force checkpoints in production for experimentation.
