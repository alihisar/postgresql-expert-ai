# Enterprise PostgreSQL Expert AI — Complete Bundle

This bundle contains every ZIP package produced so far and a consolidated project tree.

## Contents

### `archives/`

Original release packages, preserved separately:

- `postgresql-expert-ai-starter.zip`
- `postgresql-expert-ai-update-01.zip`
- `postgresql-expert-ai-foundation-update.zip`
- `postgresql-ai-training-pack-01.zip`
- `postgresql-expert-ai-github-starter.zip`
- `enterprise-postgresql-expert-ai-v0.1.0.zip`
- `enterprise-postgresql-expert-ai-v0.2.0.zip`

### `consolidated-project/`

A merged working tree created by applying the packages in chronological order. Later releases override overlapping files from earlier releases.

## Recommended Git Workflow

Copy the contents of `consolidated-project/` into your local repository, then run:

```powershell
git status
git add .
git commit -m "chore: consolidate PostgreSQL expert AI training releases through v0.2.0"
git push origin main
```

## Important Note

The project is still in early curriculum development. The existing files establish the training framework and the first competencies. Later releases will add deeper PostgreSQL internals, query processing, performance, HA, operations, backup, recovery, and upgrade material.
