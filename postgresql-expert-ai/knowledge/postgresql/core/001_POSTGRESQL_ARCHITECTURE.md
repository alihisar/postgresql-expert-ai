---
id: pg-core-architecture-001
title: PostgreSQL Architecture and Execution Flow
category: PostgreSQL Core
difficulty: Intermediate
language: English
versions:
  - 14
  - 15
  - 16
  - 17
  - 18
tags:
  - architecture
  - process-model
  - memory
  - wal
  - storage
  - executor
learning_objectives:
  - Explain the PostgreSQL process model
  - Trace a query from client connection to result
  - Trace a durable transaction from SQL to WAL flush
  - Distinguish shared memory, local memory, operating-system cache, and storage
  - Diagnose failures by architectural layer
---

# PostgreSQL Architecture and Execution Flow

## 1. Why architecture matters

A DBA cannot diagnose PostgreSQL correctly by memorizing commands alone. Every incident is easier to understand when the DBA can answer four questions:

1. Which process is doing the work?
2. Which memory area is involved?
3. Which storage structure is being read or modified?
4. Which background process is responsible for persistence or maintenance?

This chapter teaches the execution flow that connects SQL, memory, WAL, data files, background processes, and client sessions.

## 2. High-level architecture

```text
Client Application
      |
      v
Postmaster / Main Server Process
      |
      +---- Backend Process for Session A
      +---- Backend Process for Session B
      +---- Backend Process for Session C
      |
      +---- Background Processes
             |- Checkpointer
             |- Background Writer
             |- WAL Writer
             |- Autovacuum Launcher
             |- Autovacuum Workers
             |- Archiver
             |- Logical Replication Launcher

Shared Memory
  |- shared_buffers
  |- WAL buffers
  |- lock tables
  |- process state
  |- transaction state

Operating System
  |- page cache
  |- filesystem
  |- process scheduler
  |- network stack

Persistent Storage
  |- table and index files
  |- WAL files
  |- control files
  |- configuration and metadata
```

## 3. The process model

PostgreSQL uses a process-based architecture. A client connection is normally served by a dedicated backend process.

### Why a process model?

A separate backend process provides isolation. A failure in one backend is less likely to corrupt another backend's private memory. The operating system also provides mature scheduling, accounting, and protection between processes.

### Important consequence

A large number of direct client connections means a large number of backend processes. Each process consumes memory and scheduling resources. This is one reason connection pooling matters.

### Production example

Assume:

- 2,000 client connections
- 500 active queries
- `work_mem = 64MB`
- several sort and hash nodes per query

It would be incorrect to estimate memory as only `2,000 × 64MB`, because `work_mem` is not simply allocated once per connection. It can be used by multiple plan nodes, while inactive sessions may use much less. The correct conclusion is that high connection count plus high concurrency plus high per-node memory can create severe memory pressure.

## 4. Connection flow

```text
Client opens TCP connection
        ↓
Postmaster accepts connection
        ↓
Authentication and pg_hba.conf checks
        ↓
Backend process is created
        ↓
Session parameters are initialized
        ↓
Client sends SQL
        ↓
Parser → Rewriter → Planner → Executor
        ↓
Rows or command status returned to client
```

### Failure interpretation

- Authentication error: inspect `pg_hba.conf`, role state, TLS, and password method.
- Connection refused: inspect listening address, port, firewall, service state.
- Too many connections: inspect `max_connections`, pool design, and leaked sessions.
- Long connection time: inspect DNS, TLS, authentication, process pressure, and network latency.

## 5. SQL processing pipeline

### 5.1 Parser

The parser checks SQL syntax and builds a parse tree.

### 5.2 Rewriter

The rewriter expands rules and views.

### 5.3 Planner/Optimizer

The planner evaluates alternative execution paths using statistics and cost estimates.

### 5.4 Executor

The executor runs the selected plan and produces rows.

```text
SQL text
  ↓
Parse tree
  ↓
Rewritten query tree
  ↓
Planned execution tree
  ↓
Executor nodes
  ↓
Result rows
```

### Production diagnostic lesson

A slow query can fail at different conceptual stages:

- Planning is slow because there are many partitions or joins.
- Planning chooses a poor plan because estimates are wrong.
- Execution is slow because too much data is processed.
- Execution waits on I/O, locks, CPU, or temporary files.

Therefore always compare `Planning Time` and `Execution Time`.

## 6. Memory architecture

PostgreSQL uses several distinct memory layers.

### Shared memory

Shared across PostgreSQL processes.

Important examples:

- `shared_buffers`
- WAL buffers
- lock management structures
- shared process state

### Backend-local memory

Private to a backend process.

Important examples:

- query execution context
- sort and hash memory
- parser and planner memory
- session state

### Operating-system cache

PostgreSQL data files are also cached by the operating system. This explains why a PostgreSQL `shared read` does not guarantee a physical storage read.

### Storage

Persistent files on SSD, NVMe, SAN, local disk, cloud block storage, or another supported medium.

### Key lesson

```text
PostgreSQL shared buffer miss
          ↓
Read requested from operating system
          ↓
OS page cache hit OR physical storage read
```

Therefore:

- `shared hit` means the PostgreSQL buffer cache already contained the page.
- `shared read` means PostgreSQL had to obtain the page through the file layer.
- only additional evidence can prove physical device I/O.

## 7. Shared buffers and the buffer manager

PostgreSQL reads table and index pages into `shared_buffers` before normal processing.

A typical page size is 8 KiB, although builds can differ.

### Example calculation

```text
shared read = 50,000 pages
50,000 × 8 KiB = 400,000 KiB ≈ 390.6 MiB
```

This is page traffic, not necessarily user-visible result size.

### Why buffer hits still cost resources

A buffer hit avoids obtaining the page through the file layer, but PostgreSQL still performs work:

- buffer lookup
- pinning
- visibility checks
- tuple examination
- predicate evaluation
- memory access
- result construction

A query with millions of buffer hits can still be CPU-bound.

## 8. WAL and durability flow

WAL stands for Write-Ahead Logging. PostgreSQL must write the corresponding WAL record before the changed data page is allowed to be written to persistent storage.

```text
INSERT / UPDATE / DELETE
          ↓
Backend modifies a page in shared_buffers
          ↓
Backend generates WAL records
          ↓
WAL records enter WAL buffers
          ↓
WAL is flushed according to commit semantics
          ↓
Commit becomes durable
          ↓
Dirty data pages may be written later
```

### Critical principle

The commit does not normally wait for every modified table page to be written. It waits for the required WAL durability condition.

### Production scenario

A system has fast table-file writes but slow WAL storage. Commit latency can still be high because durable commits depend on WAL flush latency.

### Evidence to inspect

- `pg_stat_wal`
- WAL write and sync timings when available
- storage latency
- `synchronous_commit`
- synchronous replication state
- checkpoint frequency
- WAL volume per transaction

## 9. Checkpointer and background writer

### Checkpointer

The checkpointer creates a recoverable point and writes dirty buffers according to checkpoint requirements.

Too-frequent checkpoints can cause:

- write bursts
- increased WAL volume due to full-page images
- latency spikes

### Background writer

The background writer attempts to write reusable dirty buffers gradually, reducing the number of writes that foreground backends must perform.

### Important distinction

The background writer does not replace checkpoints. These processes have related but different responsibilities.

## 10. Autovacuum architecture

The autovacuum launcher starts autovacuum workers. Workers perform VACUUM and ANALYZE work according to thresholds and configuration.

Autovacuum is required for more than space cleanup. It is essential for:

- removing dead tuple versions when possible
- maintaining visibility information
- updating planner statistics through auto-analyze
- preventing transaction ID wraparound

### Dangerous misunderstanding

Disabling autovacuum to reduce I/O is not a safe long-term optimization. It can create bloat, stale statistics, and wraparound risk.

## 11. Storage layout

A PostgreSQL cluster contains databases, relations, WAL, control data, and metadata.

Important structures include:

- base relation files
- tablespace directories
- WAL directory
- visibility map forks
- free space map forks
- initialization forks for unlogged relations

### Relation forks

A relation may have:

- main fork: table or index contents
- FSM fork: free-space information
- VM fork: visibility information
- init fork: initial state for unlogged relations

## 12. Page and tuple concepts

PostgreSQL stores table data in pages. A page contains headers, line pointers, tuples, and free space.

A tuple contains metadata such as transaction visibility information.

This architecture explains:

- why updates create new tuple versions
- why VACUUM is needed
- why index-only scans depend on visibility information
- why bloat occurs

## 13. MVCC in the architecture

MVCC allows readers and writers to operate with reduced blocking by maintaining multiple row versions.

```text
Old row version remains visible to older snapshot
New row version becomes visible to newer snapshot
VACUUM later removes obsolete versions when safe
```

### Production implication

A long-running transaction can retain old row versions, increase bloat, delay cleanup, and increase storage pressure.

## 14. End-to-end read example

Query:

```sql
SELECT id, total_amount
FROM orders
WHERE customer_id = 42;
```

Possible flow:

```text
Client sends query
  ↓
Backend parses and plans
  ↓
Planner chooses Index Scan
  ↓
Executor reads index pages
  ↓
Matching tuple locations are found
  ↓
Heap pages are read if required
  ↓
MVCC visibility is checked
  ↓
Rows are returned
```

### Diagnostic questions

- Was the index used?
- How many index and heap pages were touched?
- Were heap fetches required?
- Were estimates accurate?
- Was the result selective enough for an index?

## 15. End-to-end write example

Transaction:

```sql
BEGIN;
UPDATE accounts
SET balance = balance - 100
WHERE id = 10;
COMMIT;
```

Simplified flow:

```text
Backend finds tuple
  ↓
New tuple version is created
  ↓
Indexes may be updated
  ↓
WAL records are generated
  ↓
Dirty pages remain in shared_buffers
  ↓
Commit WAL is flushed as required
  ↓
Client receives commit success
  ↓
Data pages are written later
```

## 16. Production incident walkthrough

### Symptom

Users report that commits suddenly take 200 ms instead of 5 ms.

### Incorrect reaction

Increase `shared_buffers` immediately.

### Correct reasoning

1. Commit latency is strongly related to WAL durability.
2. Inspect WAL write and sync latency.
3. Inspect synchronous replication state.
4. Inspect storage latency and queue depth.
5. Check whether a checkpoint or storage event coincides with the spike.
6. Verify whether `synchronous_commit` or synchronous standby configuration changed.

### Lesson

Architecture prevents category errors. A commit-latency issue is not automatically a buffer-cache issue.

## 17. Common misconceptions

### “PostgreSQL writes table pages before commit”

Normally false. WAL durability is the key commit requirement.

### “A buffer hit is free”

False. It avoids file-layer acquisition but still consumes CPU and memory bandwidth.

### “Autovacuum is optional maintenance”

False. It is a core correctness and sustainability mechanism.

### “HAProxy decides the PostgreSQL primary”

False. HAProxy routes traffic based on health checks; Patroni and the DCS coordinate role state.

### “A PostgreSQL read always means disk I/O”

False. The OS page cache may satisfy the read.

## 18. Diagnostic decision tree

```text
PostgreSQL performance problem
  |
  +-- Planning time high?
  |      +-- Yes: inspect partitions, joins, prepared statements, statistics
  |
  +-- Execution time high?
         |
         +-- Waiting?
         |     +-- Lock → inspect blockers
         |     +-- I/O → inspect page reads and storage latency
         |     +-- WAL → inspect WAL flush and replication
         |
         +-- CPU high?
         |     +-- excessive rows, functions, joins, JSON, JIT
         |
         +-- Temp I/O?
               +-- sort/hash spill, work_mem, plan shape
```

## 19. Hands-on lab

### Goal

Observe backend processes, buffer behavior, and WAL activity.

### Step 1: identify the backend

```sql
SELECT pg_backend_pid();
```

### Step 2: create a test table

```sql
CREATE TABLE architecture_lab (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    payload text NOT NULL,
    created_at timestamptz NOT NULL DEFAULT now()
);
```

### Step 3: generate rows

```sql
INSERT INTO architecture_lab (payload)
SELECT repeat(md5(g::text), 4)
FROM generate_series(1, 100000) AS g;
```

### Step 4: inspect relation size

```sql
SELECT
    pg_size_pretty(pg_relation_size('architecture_lab')) AS heap,
    pg_size_pretty(pg_indexes_size('architecture_lab')) AS indexes,
    pg_size_pretty(pg_total_relation_size('architecture_lab')) AS total;
```

### Step 5: inspect plan and buffers

```sql
EXPLAIN (ANALYZE, BUFFERS, WAL, SETTINGS)
SELECT *
FROM architecture_lab
WHERE id = 50000;
```

### Step 6: update and observe WAL

```sql
EXPLAIN (ANALYZE, BUFFERS, WAL)
UPDATE architecture_lab
SET payload = payload || 'x'
WHERE id BETWEEN 100 AND 200;
```

### Questions

1. Which process executed the statement?
2. Which pages were read or hit?
3. Did the update generate WAL?
4. Did the commit require the heap pages to be written immediately?
5. Which background processes may later write dirty pages?

## 20. Quiz

### Question 1

What does `shared read=1000` prove?

A. Exactly 1000 physical disk reads occurred.
B. PostgreSQL obtained 1000 pages through the file layer because they were not already in shared buffers.
C. The query returned 1000 pages to the client.
D. The operating system cache was bypassed.

**Answer: B**

### Question 2

Why can a query with only buffer hits still be slow?

A. Buffer hits always trigger WAL.
B. Buffer hits still require tuple processing, visibility checks, predicate evaluation, and memory access.
C. Buffer hits force a checkpoint.
D. Buffer hits disable indexes.

**Answer: B**

### Question 3

Which mechanism primarily protects transaction durability at commit?

A. Immediate heap-page write
B. Autovacuum
C. Required WAL flush
D. Visibility map update

**Answer: C**

## 21. AI reasoning exercise

### Input

```text
Users report high commit latency.
shared_buffers hit ratio is 99.9%.
WAL sync latency increased from 1 ms to 80 ms.
```

### Expected reasoning

- High buffer hit ratio does not explain commit latency.
- WAL sync latency directly affects durable commit latency.
- Investigate WAL storage, synchronous replication, checkpoint overlap, and I/O queueing.
- Do not recommend increasing shared buffers as the first action.

## 22. Key lessons

1. PostgreSQL is a coordinated system of backend processes, shared memory, background processes, WAL, and persistent storage.
2. Query diagnosis requires distinguishing planning, execution, waiting, CPU, memory, I/O, and WAL.
3. WAL durability and table-page persistence are related but not identical.
4. MVCC, VACUUM, visibility, and storage layout are connected.
5. Correct diagnosis begins by identifying the architectural layer that owns the observed behavior.
