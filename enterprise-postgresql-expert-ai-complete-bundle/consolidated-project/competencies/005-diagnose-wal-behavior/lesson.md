# Diagnose WAL Behavior

## Capability

After this competency, the AI can explain WAL behavior, identify why WAL grows, and classify WAL-related production risk.

## First Principles

Durability requires a persistent record of change.

PostgreSQL writes a sequential record before modified data pages must reach their final relation files.

That record is WAL.

## Mental Model

```text
Transaction changes data
        |
        v
WAL record generated
        |
        v
WAL buffer
        |
        v
WAL flushed according to commit semantics
        |
        v
Dirty data page may be written later
```

## Why WAL Exists

WAL supports:

- crash recovery
- replication
- PITR
- archive recovery
- durability guarantees

## WAL Is Not the Table

WAL contains change records, not a normal queryable copy of the database.

## Core Terms

- WAL record
- WAL segment
- LSN
- flush
- replay
- archive
- timeline

## Production Case 1 — WAL Growth

Symptoms:

- `pg_wal` grows continuously
- replication lag is high
- archive queue grows
- disk space falls

Possible causes:

- failed `archive_command`
- inactive replication slot
- lagging replica
- very high write workload
- long-running backup
- retention settings

The correct first action is not to delete WAL files.

The correct first action is to identify the retention reason.

## Evidence Sources

```sql
SELECT * FROM pg_stat_wal;

SELECT slot_name, active, restart_lsn, confirmed_flush_lsn
FROM pg_replication_slots;

SELECT application_name, state, sent_lsn, write_lsn, flush_lsn, replay_lsn
FROM pg_stat_replication;
```

## Wrong Thinking

- "WAL growth means PostgreSQL is broken."
- "Delete old WAL files manually."
- "WAL generation is always a problem."
- "Replication lag only affects replicas."

## Trade-offs

Higher WAL generation may be expected during:

- bulk load
- index creation
- vacuum full
- heavy updates
- logical replication
- maintenance

Risk depends on:

- free disk
- archive health
- replica health
- retention source
- recovery objectives
