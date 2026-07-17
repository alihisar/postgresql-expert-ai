---
id: framework-reasoning-001
title: Evidence-Based DBA Reasoning Engine
category: AI Framework
difficulty: Core
language: English
version: 1.0.0
tags:
  - reasoning
  - diagnosis
  - root-cause-analysis
  - risk
---

# Evidence-Based DBA Reasoning Engine

This document teaches the model how to reason, not merely what to remember.

## 1. The DBA reasoning loop

Use this loop for every incident or technical question:

```text
Observe
  ↓
Classify
  ↓
Collect evidence
  ↓
Build hypotheses
  ↓
Test hypotheses
  ↓
Assess risk
  ↓
Choose the safest action
  ↓
Validate
  ↓
Prevent recurrence
```

## 2. Separate symptom, mechanism, and root cause

Example:

- Symptom: queries are slow.
- Mechanism: sessions are waiting on `DataFileRead`.
- Root cause: a plan regression caused a 500 GB table scan after stale statistics.

Do not confuse these levels.

## 3. Layer classification

Classify every problem into one or more layers:

| Layer | Typical evidence |
|---|---|
| SQL | query text, parameters, row counts |
| Planner | `EXPLAIN`, estimates, statistics |
| Executor | actual rows, loops, temp files |
| Storage | I/O latency, queue depth, throughput |
| Memory | OOM, swap, temp spill, cache pressure |
| WAL | WAL rate, archive delay, checkpoints |
| Replication | LSN positions, lag, slot state |
| Patroni | role, leader lock, timeline, REST state |
| Consul | quorum, Raft leader, sessions, ACL |
| HAProxy | health checks, backend state, routing |
| Keepalived | VRRP state, VIP, track scripts |
| Linux | CPU, memory, disk, network, systemd |

## 4. Hypothesis testing

For each hypothesis, write:

- hypothesis
- evidence that supports it
- evidence that contradicts it
- evidence still required
- confidence level

Example:

```text
Hypothesis: replication lag is caused by network bandwidth.
Supports: receive LSN advances slowly and network saturation is present.
Contradicts: WAL receiver reports normal receive rate.
Required: network throughput, sender WAL rate, receiver write/replay rate.
Confidence: low until evidence is collected.
```

## 5. Risk-first action selection

Prefer actions in this order:

1. read-only inspection
2. session-local testing
3. reversible configuration change
4. controlled restart
5. switchover
6. failover
7. rebuild or destructive recovery

The model must not jump directly to restart, reinit, or failover.

## 6. Validation contract

Every action must include expected evidence.

Bad:

```text
Create the index and test again.
```

Good:

```text
Create the index concurrently, run ANALYZE, and compare:
- execution time
- actual rows
- shared read
- rows removed by filter
- plan node type
If the plan does not change, inspect selectivity and statistics before keeping the index.
```

## 7. Teaching mechanism

When explaining a system, use this pattern:

```text
Purpose → Components → Flow → Failure mode → Evidence → Safe response
```

Example for WAL:

```text
Purpose: durability and recovery.
Components: backend, WAL buffers, WAL writer, storage, archiver.
Flow: change creates WAL record, WAL is flushed, commit becomes durable.
Failure mode: archive backlog.
Evidence: archive status, WAL directory growth, logs.
Safe response: restore archive throughput; do not delete WAL manually.
```
