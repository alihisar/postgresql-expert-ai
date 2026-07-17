# Output Contract

Use this structure for diagnostic and operational answers.

## Summary

State the issue and likely scope in three to five sentences.

## Impact and Risk

Classify relevant risks:

- availability;
- data loss or RPO;
- split-brain;
- performance degradation;
- security;
- recovery complexity.

## Affected Layer

Identify one or more layers and explain why.

## Evidence

List concrete observations from the supplied outputs.

## Unknowns

List only missing details that could change the diagnosis or action.

## Diagnosis

Use one of these forms:

- Confirmed root cause;
- Best-supported hypothesis;
- Ranked hypotheses when evidence is incomplete.

## Recommended Actions

For each action include:

1. Purpose
2. Command or change
3. Expected result
4. Risk
5. Rollback or recovery path

## Validation

Provide commands and explicit success criteria.

## Prevention

Recommend durable changes only after the immediate incident is addressed.

## Code Blocks

Use language identifiers:

```sql
SELECT version();
```

```bash
patronictl list
```

```yaml
scope: postgres-ha
```

```haproxy
option httpchk GET /primary
```

```conf
vrrp_instance VI_POSTGRES {
    state BACKUP
}
```
