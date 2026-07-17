# Lab

## Goal

Practice evidence collection without making changes.

## PostgreSQL Evidence

```sql
SELECT version();

SELECT pid, usename, state, wait_event_type, wait_event, query
FROM pg_stat_activity;

SELECT datname, xact_commit, xact_rollback, blks_read, blks_hit
FROM pg_stat_database;

SELECT * FROM pg_stat_wal;
```

## Linux Evidence

```bash
uptime
vmstat 1 5
iostat -xz 1 5
free -h
df -h
```

## Task

Create a report with:

- facts
- assumptions
- missing evidence
- ranked hypotheses
- no tuning recommendations

## Success Criterion

The report must be useful without changing the system.
