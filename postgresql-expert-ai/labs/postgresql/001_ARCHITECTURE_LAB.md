---
id: lab-pg-001
title: PostgreSQL Architecture Observation Lab
category: Lab
language: English
version: 1.0.0
---

# PostgreSQL Architecture Observation Lab

## Objective

Observe backend identity, memory-visible behavior, relation storage, buffer activity, and WAL generation.

## Safety

Run this lab in a disposable local database or test environment.

## Tasks

1. Create the test table from the architecture chapter.
2. Identify your backend PID.
3. Inspect the session in `pg_stat_activity`.
4. Run read queries with `EXPLAIN (ANALYZE, BUFFERS)` twice.
5. Compare cold and warm execution behavior.
6. Run an update with `WAL` instrumentation.
7. Inspect table and index sizes.
8. Explain why commit durability does not require immediate heap-page persistence.

## Expected learning outcome

The learner should be able to trace one query through backend execution, buffer access, MVCC visibility, WAL generation, and later persistence.
