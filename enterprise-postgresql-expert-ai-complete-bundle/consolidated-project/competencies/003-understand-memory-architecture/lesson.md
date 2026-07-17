# Understand PostgreSQL Memory Architecture

## Capability

After this competency, the AI can explain PostgreSQL memory areas and evaluate memory tuning risks.

## Memory Categories

### Shared Memory

Used across server processes.

Examples:

- shared buffers
- WAL buffers
- lock structures
- shared metadata

### Per-Backend Memory

Private to a backend.

Examples:

- executor contexts
- sort memory
- hash memory
- query parsing and planning memory

### Maintenance Memory

Used by maintenance operations such as VACUUM and index creation.

### Operating-System Memory

Includes page cache and other kernel-managed memory.

## Critical Principle

`work_mem` is not a simple per-server allocation.

It may be consumed per operation and multiplied by:

- several nodes in one plan
- parallel workers
- concurrent sessions

Example:

```text
100 active queries
× 3 memory-consuming nodes
× 64 MB
= 19,200 MB potential allocation
```

This is a risk model, not a guaranteed allocation.

## Wrong Thinking

- "16 GB RAM means shared_buffers should be 12 GB."
- "No swap usage means memory is healthy."
- "A large work_mem value only affects one query."
- "OS page cache is wasted memory."

## Production Story

A team increased `work_mem` from 4 MB to 256 MB globally.

Single-query performance improved.

During peak concurrency, the operating system invoked the OOM killer.

The correct lesson:

Tune with concurrency, plan shape, and workload distribution in mind.
