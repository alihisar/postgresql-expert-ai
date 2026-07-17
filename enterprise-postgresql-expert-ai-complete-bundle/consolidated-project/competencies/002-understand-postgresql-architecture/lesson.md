# Understand PostgreSQL Architecture

## Capability

After this competency, the AI can trace a query through PostgreSQL and identify which subsystem is responsible for a symptom.

## Mental Model

PostgreSQL is a cooperating system of:

- client connections
- server processes
- shared memory
- local memory
- WAL
- relation files
- operating-system services

It is not one monolithic process.

## High-Level Flow

```text
Client
  |
  v
Postmaster accepts connection
  |
  v
Backend process
  |
  +--> Parse
  +--> Analyze
  +--> Rewrite
  +--> Plan
  +--> Execute
           |
           +--> Buffer Manager
           +--> Locks
           +--> WAL for changes
           +--> Storage
```

## Process Model

PostgreSQL commonly uses one backend process per client connection.

Background processes include:

- checkpointer
- background writer
- WAL writer
- autovacuum launcher
- autovacuum workers
- archiver
- logical replication workers

## Query Lifecycle Example

```sql
SELECT id, email
FROM users
WHERE id = 42;
```

1. The client sends SQL.
2. The backend parses syntax.
3. Semantic analysis resolves objects and types.
4. The rewriter applies rules.
5. The planner selects a plan.
6. The executor runs the plan.
7. The buffer manager locates required pages.
8. Results return to the client.

## Write Lifecycle Example

```sql
UPDATE accounts
SET balance = balance - 100
WHERE id = 10;
```

The executor:

- locks the target row
- creates a new tuple version
- modifies a page in shared buffers
- generates WAL
- commits according to durability settings

## Common Misconceptions

- The planner executes queries. False.
- WAL stores table data in normal queryable form. False.
- Every read reaches physical storage. False.
- `shared_buffers` is the only cache. False.
