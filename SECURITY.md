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
- Related product URLs on `https://domainscan.tools` (docs, reports and signup links returned by the server)

The plugin connects to the MCP endpoint. The remote service also contacts public DNS/TLS/HTTP targets and discovery sources; subdomain discovery uses crt.sh / crt.name, and reverse-IP discovery uses public certificate and reverse-IP sources. A single configured MCP URL does not mean the backend never contacts third parties.

## Trust boundary

| Surface | Risk | Mitigation |
|---|---|---|
| Plugin package | Supply-chain / local RCE | No executable code; MIT-licensed config + docs only |
| Anonymous `scan_domain` | Recon / cost abuse | Server-side rate limits, SSRF guards, input validation (server-owned) |
| Anonymous `reverse_ip_lookup` / `find_subdomains` | Recon / external queries / cached or incomplete data | Server-owned lookup limits and validation; no scan reports or scan-quota consumption; explain source coverage and truncation |
| Anonymous `get_monitoring_signup_link` | Misleading claims of activation | Only returns a pre-filled registration link; no monitoring activation, account creation or email |
| Account tools | Data exposure | Require `Authorization: Bearer ds_live_...`; keys hashed server-side, revocable |
| Report links `/r/{token}` | Leak of findings | Opaque 128-bit tokens, short TTL, robots Disallow `/r/` |

The backend mitigations above are documented by the service operator. This repository contains the client configuration and guidance, not the MCP server implementation. A pinned plugin commit fixes the installed configuration and skill; it does not freeze the remote service's implementation or tool behavior.

Report URLs act as bearer links: anyone who obtains a URL can access its report until expiry. Opaque tokens reduce guessing; `noindex` and `robots.txt` concern indexing and do not provide access control. Treat these URLs as sensitive.

Scans persist reports, consume quota and may trigger irreversible administrative notifications. Lookup calls query external sources, cache results and count usage. Account queries log usage and update key timestamps. These side effects must not be described as zero logging or as purely local operations.

## Discovery verification: 2026-10-31

- `grok plugin validate .` passed for package `1.2.0` with Grok Build `1.0.24`, recognizing one skill directory and the remote MCP configuration.
- An anonymous `initialize` request returned server `domain-scan` version `1.2.0`, protocol `2025-03-26`.
- Anonymous `tools/list` returned exactly four tools: `scan_domain`, `reverse_ip_lookup`, `find_subdomains`, and `get_monitoring_signup_link`.
- `scan_domain` advertises a 7-day report lifetime and annotations `readOnlyHint: false`, `destructiveHint: true`, `idempotentHint: false`, `openWorldHint: true`.
- Both lookup tools advertise `readOnlyHint: false`, `destructiveHint: false`, `idempotentHint: true`, `openWorldHint: true`, consistent with external queries, caching and usage counting.
- The signup-link tool advertises `readOnlyHint: true`, `destructiveHint: false`, `idempotentHint: true`, `openWorldHint: false` and explicitly does not activate monitoring.
- Current instructions advertise anonymous scan/lookup limits and authenticated limits by plan, as recorded in README.md.

This was a discovery-only check, not a new scan, lookup, authenticated-account test or backend audit. Report expiry and rate-limit enforcement were not retested. The historical checks below describe the older service and do not establish today's anonymous tool set or package validation.

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
- [ ] Anonymous scan/lookup limits and authenticated rate-limit enforcement match the current advertised plan limits.
- [ ] Report pages emit `noindex` and become inaccessible after the advertised 7 days.
- [ ] Key storage, revocation and server-side logging match the documented behavior.
- [ ] Full Git history has been checked for secrets.

The tools do not require personal information in their arguments. This does not establish that the service processes no personal data: it receives submitted domains, request metadata and, for account tools, credentials and account-related requests. No zero-logging, regulatory-compliance or comprehensive XSS-audit claim is made by this package.

## Reporting issues

Email: hello@domainscan.tools
