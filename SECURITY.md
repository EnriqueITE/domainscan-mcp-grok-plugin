# Security model: domainscan-mcp-grok-plugin

## What this plugin is

A **thin remote MCP pointer**. It does **not** ship:

- local binaries / `npx` / shell commands
- hooks that run on agent events
- postinstall scripts
- embedded API keys or secrets
- filesystem or browser automation

Network destinations and related links:

- `https://api.domainscan.tools/functions/v1/mcp-server` (MCP)
- Related product URLs on `https://domainscan.tools` (docs / report links returned by the server)

## Trust boundary

| Surface | Risk | Mitigation |
|---|---|---|
| Plugin package | Supply-chain / local RCE | No executable code; MIT-licensed config + docs only |
| Anonymous `scan_domain` | Recon / cost abuse | Server-side rate limits, SSRF guards, input validation (server-owned) |
| Account tools | Data exposure | Require `Authorization: Bearer ds_live_...`; keys hashed server-side, revocable |
| Report links `/r/{token}` | Leak of findings | Opaque 128-bit tokens, short TTL, robots Disallow `/r/` |

The backend mitigations above are documented by the service operator. This repository contains the client configuration and guidance, not the MCP server implementation. A pinned plugin commit fixes the installed configuration and skill; it does not freeze the remote service's implementation or tool behavior.

Report URLs act as bearer links: anyone who obtains a URL can access its report until expiry. Opaque tokens reduce guessing; `noindex` and `robots.txt` concern indexing and do not provide access control. Treat these URLs as sensitive.

## Verification record: 2026-09-09

Package validation covers plugin version `1.1.1`. The live endpoint checks below were performed on 2026-09-09 against server version `1.1.0`; the package and hosted server have independent versions.

- `grok plugin validate .` passed with Grok Build 0.2.118.
- xAI's component extractor recognized one skill and one HTTP MCP server.
- `initialize` returned HTTP 200, protocol `2025-03-26`, server `domain-scan` version `1.1.0`.
- `notifications/initialized` returned HTTP 202 with no response body.
- Anonymous `tools/list` returned only `scan_domain`.
- An unauthenticated `list_domains` call returned an API-key-required error and no account data.
- One `scan_domain` call for `domainscan.tools` returned a score, findings and an opaque report link. The link is intentionally omitted here.

The local Grok MCP doctor discovered the configuration but did not start the connection because the temporary checkout was not trusted. Installation and use inside a Grok conversation were not verified.

The existing operator checklist reported a successful anonymous rate-limit test on 2026-09-09 (20 scans per 10 minutes, fail-closed). That test was not repeated during this publication review, and its raw evidence is not included in this repository.

## Backend checks still to verify

- [ ] Private / loopback IPs and DNS-to-private targets remain rejected.
- [ ] Invalid API keys are rejected across account tools; valid keys isolate different users' data.
- [ ] Authenticated rate-limit enforcement matches the advertised 100 calls/hour.
- [ ] Report pages emit `noindex` and become inaccessible after the advertised 48 hours.
- [ ] Key storage, revocation and server-side logging match the documented behavior.
- [ ] Full Git history has been checked for secrets.

The tools do not require personal information in their arguments. This does not establish that the service processes no personal data: it receives submitted domains, request metadata and, for account tools, credentials and account-related requests. No zero-logging, regulatory-compliance or comprehensive XSS-audit claim is made by this package.

## Reporting issues

Email: hello@domainscan.tools
