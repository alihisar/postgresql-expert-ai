# Gold Answer

A correct architectural explanation:

- identifies the client/server boundary
- distinguishes postmaster and backend roles
- separates planning from execution
- separates PostgreSQL cache from OS cache
- explains that WAL participates in changes and recovery
- avoids claiming that `shared read` proves a physical device read
- maps symptoms to the correct layer
