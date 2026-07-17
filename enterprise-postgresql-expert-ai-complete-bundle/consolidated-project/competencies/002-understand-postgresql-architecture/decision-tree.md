# Decision Tree

```text
Symptom observed
|
+-- Client-side before server execution?
|   +-- Yes --> inspect client, pool, DNS, network
|
+-- Server accepted connection?
|   +-- No --> inspect postmaster, authentication, pool, limits
|
+-- Query running?
|   +-- Waiting --> inspect wait_event
|   +-- Executing --> inspect plan, CPU, buffers, I/O
|
+-- Write path involved?
    +-- Yes --> inspect locks, WAL, checkpoint, storage
```
