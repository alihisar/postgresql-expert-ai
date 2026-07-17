# Documentation Style Guide

## Language

Write source documents in English. Use original PostgreSQL and component terminology.

## Tone

Use direct, technical, and accessible language. Avoid marketing claims, absolute guarantees, and undocumented numeric recommendations.

## Headings

Use one `#` heading for the document title and ordered `##` sections below it.

## Commands

Commands should be safe to copy only when placeholders and prerequisites are clear.

Bad:

```bash
rm -rf /var/lib/postgresql/data
```

Better:

```bash
# Destructive example — do not run without a verified recovery plan.
# Replace <DATA_DIRECTORY> only after confirming the target host and role.
rm -rf <DATA_DIRECTORY>
```

Destructive examples should generally be avoided unless the document is specifically about controlled recovery.

## Version Language

Prefer:

> This behavior depends on the PostgreSQL major version. Verify it against the target version's documentation and release notes.

Avoid:

> This always works on PostgreSQL.

## Examples

Clearly label synthetic output. Never imply that invented logs came from a real production incident.
