---
name: domain-scan
description: Use when the user wants to check a domain's security posture: SPF, DKIM, DMARC, SSL/TLS expiry, HTTPS/HSTS/CSP, DNSSEC, CAA, BIMI, domain expiration, or DNS blocklists, or summarize Domain Scan findings.
---

# Domain Scan

## When to use

- User asks to scan, check, or audit a domain (email auth, SSL, headers, DNSSEC, reputation).
- User pastes a hostname and wants a security score / fix hints.

## How to call

1. Use the MCP tool `scan_domain` with a **bare domain** only (`example.com`).
2. Never pass `https://`, paths, ports, emails, passwords, API keys, or personal data.
3. Optional `dkim_selector` only if the user knows it.

## How to answer

- Lead with the 0-100 score and failing / warning checks.
- Include short fix hints from the tool result.
- Mention the private `/r/...` report link expires (~48h) and **must not be posted publicly**.
- Offer monitoring trial only as a secondary suggestion.

## Account tools

Tools like `list_domains`, `get_uptime`, `list_incidents`, `get_domain_report`, and `scan_my_domain` require the user's own `ds_live_...` API key. Do not invent or reuse keys. If unauthenticated, tell them to create a key in Domain Scan settings and configure the MCP header themselves.

## Abuse / safety

- Do not loop mass scans across arbitrary third-party domains.
- Respect rate-limit errors from the server.
- Do not treat report links as permanent or indexable.
