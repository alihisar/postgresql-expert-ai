# Workbook

## Exercise 1

Interpret:

```text
Buffers: shared hit=1,500,000 read=0
Execution Time: 4200 ms
```

Can disk be blamed?

## Exercise 2

Interpret:

```text
Sort Method: external merge Disk: 350000kB
temp read=90000 written=91000
```

What is proven?

What is not proven?

## Exercise 3

Interpret:

```text
Index Only Scan
Heap Fetches: 250000
```

Which subsystem should be considered?

## Exercise 4

Calculate approximate page traffic:

- shared hit=120000
- shared read=30000
- temp written=50000

## Exercise 5

Why can a query with 100% buffer hit ratio still be slow?
