# Think Like a PostgreSQL DBA

An expert DBA does not start with a fix.

An expert DBA starts with a model of the system.

The model contains:

- processes
- memory
- storage
- WAL
- MVCC
- locks
- planner
- executor
- operating system
- network
- workload
- failure domains

The first question is not:

> What should I change?

The first questions are:

- What is directly observed?
- Which layer can produce this symptom?
- Which evidence is missing?
- What is the safest way to reduce uncertainty?
