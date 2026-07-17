# Decision Tree

```text
Checkpoint-related latency
|
+-- Are checkpoints frequent?
|   +-- Yes --> timed or requested?
|
+-- Are writes slow?
|   +-- Yes --> correlate storage latency
|
+-- Is WAL generation high?
|   +-- Yes --> inspect workload
|
+-- Are dirty buffers accumulating?
    +-- Yes --> inspect pacing, background writing, storage
```
