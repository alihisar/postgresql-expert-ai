# Decision Tree

```text
Dead tuples or bloat
|
+-- Long-running transactions?
|   +-- Yes --> identify owners and safe termination plan
|
+-- Autovacuum running?
|   +-- No --> inspect thresholds, cost, blocking
|
+-- High update churn?
|   +-- Yes --> inspect HOT ratio, indexes, fillfactor
|
+-- Wraparound risk?
    +-- Yes --> critical incident procedure
```
