# Decision Tree

```text
WAL growth
|
+-- Is write workload unusually high?
|   +-- Yes --> quantify source of WAL
|
+-- Is archiving failing?
|   +-- Yes --> fix archive pipeline
|
+-- Is a replication slot retaining WAL?
|   +-- Yes --> identify owner and safety of removal
|
+-- Is a replica lagging?
|   +-- Yes --> investigate network, disk, replay
|
+-- Is free disk critical?
    +-- Yes --> emergency risk plan, no manual WAL deletion
```
