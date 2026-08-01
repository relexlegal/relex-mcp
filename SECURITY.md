# Security

Relex lets you use any MCP-connected agent on legal matters **without exposing
client PII to the model**.

## Authentication

- Hosted MCP server: `https://relex.legal/api/mcp`
- Primary auth: **OAuth 2.1 + PKCE** (Google/Apple) — browser sign-in, no key paste
- Fallback: static API key from Relex **Settings → API Keys**
- Revoke under **Settings → API Keys**; paired clients under **Settings → Agents**

## Client PII never reaches the model

- Party data is sealed client-side with a key derived from the user's PII password
- Document content is redacted client-side before upload by default
- MCP `execute` blocks endpoints that would return party/document plaintext and
  returns a deep link instead
- Agents work with de-identified labels (`[Party 1]`) and anonymized counts

## This repository

- Ships **no credentials**. It only documents the public MCP endpoint and skills.
- Tool handlers and PII guards run in the Relex backend, not in this repo.

## Reporting a vulnerability

Report privately to **security@relex.legal**. Do not open a public issue for a
suspected vulnerability.
