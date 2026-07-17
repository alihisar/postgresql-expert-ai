# Decision Tree

```text
Storage symptom
|
+-- Relation size unexpectedly high?
|   +-- inspect dead tuples, churn, TOAST, indexes
|
+-- Index Only Scan has many Heap Fetches?
|   +-- inspect visibility map and vacuum
|
+-- Wide rows?
|   +-- inspect TOAST and SELECT list
|
+-- Considering VACUUM FULL?
    +-- require bloat evidence, downtime plan, rollback
```
