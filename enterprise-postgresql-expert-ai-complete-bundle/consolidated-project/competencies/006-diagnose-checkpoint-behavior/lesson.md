# Diagnose Checkpoint Behavior

## Capability

After this competency, the AI can diagnose checkpoint pressure without confusing checkpoints with WAL flushes.

## First Principles

A checkpoint creates a known recovery boundary.

Dirty data pages must be written so recovery does not need to replay WAL indefinitely.

## Mental Model

```text
Dirty shared buffers accumulate
        |
        v
Checkpoint starts
        |
        v
Dirty pages are written over time
        |
        v
Checkpoint record marks recovery boundary
```

## Important Distinction

Checkpointing is not the same as flushing every transaction's WAL.

WAL can be flushed for commit durability independently of a checkpoint.

## Symptoms of Checkpoint Pressure

- frequent requested checkpoints
- increasing checkpoint write time
- write latency spikes
- storage saturation
- large dirty-page bursts
- repeated checkpoint log messages

## Evidence Sources

Depending on PostgreSQL version:

```sql
SELECT * FROM pg_stat_checkpointer;
```

or older views:

```sql
SELECT * FROM pg_stat_bgwriter;
```

Also inspect:

- checkpoint logs
- WAL generation
- storage latency
- dirty page pressure

## Production Story

A system had low average CPU but periodic latency spikes every few minutes.

The root cause was checkpoint I/O bursts.

Increasing random memory parameters did not help.

The successful change combined:

- better checkpoint pacing
- storage analysis
- write-workload understanding
- validation against latency spikes

## Wrong Thinking

- "Checkpoint writes WAL."
- "More frequent checkpoints are always safer."
- "Long checkpoints always mean bad storage."
- "Checkpoint problems can be solved by one parameter in isolation."
