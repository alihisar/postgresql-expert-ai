# RAG Metadata Standard

Knowledge documents should use stable metadata so they can be filtered, chunked, and evaluated consistently.

## Required Front Matter

```yaml
---
id: pg-core-wal-001
title: Write-Ahead Logging Architecture
category: PostgreSQL
subcategory: WAL
difficulty: Advanced
status: draft
last_reviewed: 2026-07-17
postgresql_versions:
  - "16"
  - "17"
  - "18"
components:
  - PostgreSQL
tags:
  - wal
  - recovery
  - replication
llm_keywords:
  - write ahead logging
  - wal segment
  - lsn
references:
  - https://www.postgresql.org/docs/current/wal-intro.html
---
```

## ID Convention

Use lowercase stable identifiers:

```text
<domain>-<topic>-<sequence>
```

Examples:

```text
pg-core-wal-001
pg-perf-explain-001
ha-patroni-dcs-001
ops-upgrade-major-001
```

## Chunking Rules

- one primary concept per heading section;
- avoid headings that contain only one sentence;
- keep commands near their explanation;
- do not split risk warnings from the action they govern;
- repeat essential version constraints within the relevant section;
- avoid relying on positional context such as “as described above.”

## Source Classification

References may be classified in prose as:

- official documentation;
- upstream source code or release notes;
- standards or RFCs;
- validated laboratory result;
- operational guidance or field practice.

Field-practice recommendations must not be presented as official defaults.
