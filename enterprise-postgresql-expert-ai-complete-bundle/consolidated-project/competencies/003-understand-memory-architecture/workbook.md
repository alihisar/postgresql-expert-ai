# Workbook

## Exercise 1

Classify:

- shared_buffers
- work_mem
- maintenance_work_mem
- WAL buffers
- OS page cache

## Exercise 2

Calculate potential memory pressure:

- 80 concurrent queries
- 2 sorts per query
- work_mem = 64 MB

Explain why the result is an upper-risk estimate rather than guaranteed usage.

## Exercise 3

A sort spills to disk.

What evidence supports a query-local work_mem test?

## Exercise 4

Why can increasing shared_buffers create trade-offs?

## Exercise 5

Which observations suggest memory pressure at the OS layer?
