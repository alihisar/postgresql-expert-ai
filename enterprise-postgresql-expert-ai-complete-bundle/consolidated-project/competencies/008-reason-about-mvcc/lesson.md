# Reason About MVCC

## Capability

After this competency, the AI can explain visibility, snapshots, tuple versions, and the operational consequences of long transactions.

## First Principles

Readers and writers should not block each other unnecessarily.

PostgreSQL achieves this using multiple row versions.

## Mental Model

```text
Original tuple version
        |
        +-- visible to older snapshot
        |
        +-- replaced by newer tuple version
                 |
                 +-- visible to newer snapshot
```

## Tuple Visibility

A tuple's visibility depends on transaction metadata and the reader's snapshot.

Important concepts:

- xmin
- xmax
- snapshot
- transaction status
- committed vs in-progress
- frozen tuples

## UPDATE Behavior

An UPDATE usually creates a new tuple version.

The old version may remain visible to older snapshots.

## DELETE Behavior

DELETE marks a tuple version as no longer visible to future snapshots after commit.

It does not immediately remove the physical tuple.

## Long Transactions

Long-running transactions can prevent cleanup because old snapshots may still need older tuple versions.

Consequences:

- dead tuple accumulation
- table and index bloat
- vacuum delay
- increased storage
- wraparound risk

## HOT Updates

A HOT update may avoid updating indexes when indexed columns do not change and page conditions permit.

## Wrong Thinking

- "UPDATE modifies a row in place."
- "DELETE physically removes the row immediately."
- "VACUUM deletes committed rows."
- "Long transactions only affect locks."
