# Understand the PostgreSQL Storage Engine

## Capability

After this competency, the AI can reason about how PostgreSQL stores tables and indexes and connect storage structures to production symptoms.

## Storage Mental Model

```text
Database
  |
  +-- Relations
       |
       +-- Heap files
       +-- Index files
       +-- FSM
       +-- Visibility Map
       +-- TOAST
```

## Heap Storage

PostgreSQL tables are commonly stored as heap relations.

Rows are stored as tuples inside pages.

A page contains:

- page header
- line pointers
- tuples
- free space

## Relation Segmentation

Large relations are split into multiple files.

Do not assume one table equals one filesystem file.

## Free Space Map

The FSM tracks where free space is available.

It helps PostgreSQL find pages suitable for inserts or updates.

## Visibility Map

The visibility map tracks pages that are all-visible or all-frozen.

It supports:

- index-only scans
- vacuum optimization
- freeze behavior

## TOAST

Large values may be compressed or stored out-of-line.

A query selecting large text or JSON values may trigger additional I/O and CPU.

## Bloat

Bloat is not simply "unused disk."

It is the result of dead tuples, page reuse patterns, update behavior, index maintenance, and vacuum effectiveness.

## Wrong Thinking

- "VACUUM always reduces file size."
- "One table equals one file."
- "Index-only scan never accesses the heap."
- "TOAST is only a compression feature."
