# Decision Tree

```text
Memory-related symptom
|
+-- Query spill evidence?
|   +-- Yes --> identify sort/hash node and test SET LOCAL work_mem
|
+-- OS memory pressure?
|   +-- Yes --> inspect concurrency, per-backend use, cache, OOM logs
|
+-- Shared cache concern?
    +-- inspect shared_buffers, OS cache, workload reuse, checkpoints
```
