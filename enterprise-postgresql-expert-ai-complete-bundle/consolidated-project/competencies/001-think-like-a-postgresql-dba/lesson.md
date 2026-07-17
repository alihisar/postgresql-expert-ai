# Think Like an Enterprise PostgreSQL DBA

## Capability

After this competency, the AI can investigate before recommending changes.

## The Core Principle

A symptom is not a root cause.

"The database is slow" is a symptom.

Possible causes include:

- lock contention
- I/O latency
- poor estimates
- bad plan choice
- connection pressure
- checkpoint activity
- autovacuum pressure
- client-side delay
- network delay

## The Expert Loop

Observe → Classify → Hypothesize → Verify → Decide → Validate → Reflect

## Facts vs Assumptions

Fact:

> `EXPLAIN (ANALYZE, BUFFERS)` shows `temp written=45000`.

Assumption:

> The server needs more RAM.

The fact proves temporary I/O occurred. It does not prove that a system-wide memory increase is safe.

## Production Story

An application became slow. The first proposal was to create an index.

Evidence later showed every slow session was waiting on the same row lock.

The index would not have solved the incident.

The expert behavior was:

1. inspect wait events
2. identify blocker
3. understand the transaction
4. resolve the blocker safely
5. prevent recurrence

## Wrong Thinking

- "Restarting will tell us whether PostgreSQL is the problem."
- "High CPU means bad SQL."
- "Sequential Scan means missing index."
- "More memory is always better."

## Decision Rules

- Request evidence before changing production.
- Prefer reversible diagnostic actions.
- Change one variable at a time.
- Validate using measurable outcomes.
