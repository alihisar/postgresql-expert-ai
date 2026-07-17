# Contributing

Thank you for contributing to the Enterprise PostgreSQL Expert AI Framework.

## Contribution Principles

Contributions must be:

- technically accurate;
- evidence-based;
- version-aware;
- safe for production readers;
- written in English;
- structured for both humans and RAG systems.

## Required Document Metadata

New knowledge documents should begin with YAML front matter:

```yaml
---
id: unique-document-id
title: Human-readable title
category: PostgreSQL
subcategory: Architecture
difficulty: Intermediate
status: draft
last_reviewed: 2026-07-17
postgresql_versions:
  - "16"
  - "17"
  - "18"
tags:
  - architecture
references:
  - https://www.postgresql.org/docs/current/
---
```

Do not claim compatibility with a version that has not been verified.

## Document Structure

Use the sections that are relevant:

1. Overview
2. Architecture or Internal Behavior
3. Evidence and Observability
4. Configuration
5. Production Guidance
6. Common Mistakes
7. Troubleshooting
8. Risk and Rollback
9. Validation
10. Checklist
11. Related Topics
12. References

## Safety Requirements

Potentially destructive commands must include:

- risk level;
- prerequisites;
- expected impact;
- rollback or recovery path;
- validation steps.

Never recommend deleting WAL, removing a PostgreSQL data directory, forcing promotion, deleting replication slots, or reinitializing a replica without explicit risk framing.

## Commit Convention

Use concise Conventional Commit-style messages:

```text
docs(postgresql): add WAL architecture guide
feat(framework): add evidence-based reasoning contract
playbook(patroni): add DCS outage response
runbook(upgrade): add pg_upgrade validation procedure
dataset(explain): add hash spill analysis case
fix(haproxy): correct Patroni health-check example
```

## Pull Requests

A pull request should explain:

- what changed;
- why it changed;
- versions tested or documented;
- safety implications;
- sources used;
- validation performed.
