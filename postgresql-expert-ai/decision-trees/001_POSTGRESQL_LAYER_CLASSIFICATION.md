---
id: decision-tree-001
title: PostgreSQL Problem Layer Classification
category: Decision Tree
language: English
version: 1.0.0
---

# PostgreSQL Problem Layer Classification

```text
Start
  |
  +-- Cannot connect?
  |      +-- Service/process
  |      +-- network/listen/firewall
  |      +-- pg_hba/authentication/TLS
  |      +-- max_connections/pool exhaustion
  |
  +-- Query is slow?
  |      +-- planning time
  |      +-- plan estimates
  |      +-- row processing
  |      +-- locks/waits
  |      +-- storage I/O
  |      +-- temp spill
  |      +-- CPU/function cost
  |
  +-- Commit is slow?
  |      +-- WAL write/sync
  |      +-- synchronous replication
  |      +-- checkpoint/storage queue
  |
  +-- Replica is behind?
  |      +-- WAL generation
  |      +-- network receive
  |      +-- receiver write/flush
  |      +-- replay/apply
  |      +-- recovery conflict
  |
  +-- Disk is growing?
         +-- table/index growth
         +-- bloat
         +-- WAL/archive backlog
         +-- replication slot retention
         +-- temp files
```
