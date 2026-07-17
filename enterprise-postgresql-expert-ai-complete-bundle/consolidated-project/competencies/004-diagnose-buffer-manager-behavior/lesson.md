# Diagnose Buffer Manager Behavior

## Capability

After this competency, the AI can interpret PostgreSQL buffer evidence without confusing it with physical device I/O.

## Mental Model

```text
Executor
  |
  v
Buffer Manager
  |
  +-- Page in shared buffers? --> shared hit
  |
  +-- Page absent? --> shared read
                         |
                         +-- OS page cache may satisfy it
                         +-- storage may satisfy it
```

## Buffer Terms

### shared hit

The page was already available in PostgreSQL shared buffers.

It still consumes CPU and memory bandwidth.

### shared read

PostgreSQL had to read the page into shared buffers.

This does not prove a physical disk read.

### shared dirtied

A page became dirty during execution.

### shared written

A dirty page was written by the backend or execution context.

### temp read / temp written

Temporary-file I/O, commonly caused by sort or hash spill.

## Page Size Approximation

PostgreSQL commonly uses 8 KB pages.

Example:

```text
shared read = 50,000
50,000 × 8 KB ≈ 390.6 MB
```

This is page traffic, not necessarily user-visible result size.

## Worked Example

```text
Seq Scan on orders
  actual rows=120 loops=1
  Rows Removed by Filter: 4,999,880
  Buffers: shared hit=12,000 read=62,000
  I/O Timings: shared read=610.000
```

Reasoning:

1. The scan processed roughly 74,000 shared pages.
2. About 484 MB was read into shared buffers.
3. High rows removed indicates inefficient filtering.
4. I/O timing supports meaningful read cost.
5. An index may be justified if the predicate is selective and types match.
6. The index recommendation remains conditional until schema and existing indexes are checked.

## Wrong Thinking

- "99% hit ratio means the query is efficient."
- "shared read means SSD read."
- "shared hit is free."
- "high buffers always means PostgreSQL needs more memory."
