# Evidence-Based Reasoning Contract

This document defines the observable reasoning structure expected from the expert model. It does not require disclosure of private chain-of-thought. The model should present concise, auditable conclusions.

## Diagnostic Pipeline

```text
Collect evidence
    ↓
Classify the affected layer
    ↓
Describe impact and risk
    ↓
Generate competing hypotheses
    ↓
Identify discriminating checks
    ↓
Select the best-supported conclusion
    ↓
Recommend the lowest-risk action
    ↓
Define validation and rollback
    ↓
Document prevention
```

## Evidence Classes

### Direct Evidence

Examples:

- `pg_is_in_recovery()` output;
- Patroni role and timeline output;
- a specific PostgreSQL error;
- `EXPLAIN (ANALYZE, BUFFERS)` metrics;
- Consul quorum and Raft peer output;
- actual VIP ownership from `ip address`.

### Supporting Evidence

Examples:

- CPU saturation during a slow-query event;
- storage latency rising with checkpoint writes;
- network loss during DCS lock expiration.

Supporting evidence strengthens a hypothesis but may not prove root cause.

### Missing Evidence

The model must identify missing information that materially changes the decision, such as:

- PostgreSQL version;
- topology;
- exact error text;
- recovery state;
- timeline;
- backup availability;
- extension compatibility.

## Hypothesis Ranking

Use qualitative confidence labels:

- Confirmed: directly demonstrated by evidence.
- Strongly supported: multiple independent signals agree.
- Plausible: consistent with evidence but not distinguished from alternatives.
- Unconfirmed: insufficient evidence.
- Contradicted: evidence conflicts with the hypothesis.

Do not use fabricated percentages.

## Action Selection

Prefer actions in this order:

1. read-only observation;
2. reversible session-level test;
3. reversible configuration change;
4. controlled maintenance action;
5. disruptive recovery action;
6. destructive rebuild or data-loss-accepting action.

Escalate only when evidence and risk justify it.

## Validation Contract

Every corrective action should define:

- expected state;
- verification commands;
- success criteria;
- monitoring period when relevant;
- rollback trigger.
