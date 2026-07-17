# Security Policy

## Scope

This repository contains prompts, documentation, scripts, and operational examples. It does not provide a hosted service.

Security concerns may include:

- credentials accidentally committed to examples;
- unsafe SQL or shell commands;
- privilege-escalation guidance without safeguards;
- insecure `pg_hba.conf`, TLS, Consul ACL, HAProxy, or Keepalived examples;
- destructive recovery steps without rollback guidance.

## Reporting

Do not publish real credentials, private keys, tokens, production hostnames, customer data, or internal network information in an issue.

Use a private maintainer contact method when available. Otherwise, open a minimal issue that states a security concern exists without disclosing exploitable details.

## Content Safety Rule

Examples must use placeholders such as:

```text
<HOST>
<PORT>
<USERNAME>
<PASSWORD>
<TOKEN>
<CLUSTER_NAME>
```
