<p align="center">
  <a href="https://domainscan.tools">
    <img src="assets/logo.png" alt="Domain Scan by CYBERACE" width="160">
  </a>
</p>

# Domain Scan: Grok Build plugin

Official thin plugin for **[Domain Scan by CYBERACE](https://domainscan.tools)**.

Scan domain security, look up domains associated with a public IPv4 address, discover subdomains from public certificate records, and get a monitoring signup link from Grok Build. These four tools work without an account or API key. Visit **[domainscan.tools](https://domainscan.tools)** for the web scanner and monitoring dashboard.

This package **only configures a remote HTTPS MCP server**. It does not install local binaries, hooks, or scripts. Grok talks to:

`https://api.domainscan.tools/functions/v1/mcp-server`

## Ownership and maintenance

Domain Scan is a CYBERACE LTD product. This official plugin is published and maintained by Enrique, CEO of CYBERACE LTD, through his personal GitHub account [EnriqueITE](https://github.com/EnriqueITE).

The source remains at [EnriqueITE/domainscan-mcp-grok-plugin](https://github.com/EnriqueITE/domainscan-mcp-grok-plugin). Questions about ownership or maintenance can be sent to hello@domainscan.tools.

This package targets **Grok Build**. Listing it in the xAI plugin marketplace does not establish availability in the general Grok chat or X.

## Security posture (read this)

- **No local code execution** from this plugin; see [SECURITY.md](./SECURITY.md).
- **Anonymous** use exposes `scan_domain`, `reverse_ip_lookup`, `find_subdomains`, and `get_monitoring_signup_link`.
- **Account tools** require your personal API key (`ds_live_...`). Never commit keys.
- Anyone holding a report link (`/r/<hex>`) can view the report until it expires. Do **not** post these links publicly.
- Rate limits and other backend controls are described separately from the checks performed in this repository; see [SECURITY.md](./SECURITY.md).

## Install (Grok Build)

Install directly from this public repository:

```bash
grok plugin install EnriqueITE/domainscan-mcp-grok-plugin --trust
```

`--trust` authorizes Grok to load the plugin's skill and remote MCP configuration. For a reproducible installation, use `EnriqueITE/domainscan-mcp-grok-plugin@<full-commit-sha>` as the source.

This repository is public and can be installed directly. Inclusion in the xAI marketplace is pending; once accepted, users with that marketplace configured can install by name:

```bash
grok plugin install domain-scan --trust
```

After installation, enable the plugin if needed with `grok plugin enable domain-scan`, then reload the Plugins tab or start a new session.

Alternatively, configure only the MCP connection manually:

```bash
# Anonymous scans, public lookups, and monitoring signup links
grok mcp add --transport http domain-scan https://api.domainscan.tools/functions/v1/mcp-server

# Optional: your account tools
grok mcp add --transport http domain-scan https://api.domainscan.tools/functions/v1/mcp-server \
  --header 'Authorization: Bearer ${DOMAINSCAN_API_KEY}'
```

The commands above use Bash syntax. The single quotes preserve the environment-variable reference for Grok to expand when it loads the configuration. Set `DOMAINSCAN_API_KEY` in your local environment; do not paste the key into chat or commit it. In PowerShell, write the authenticated command on one line without the Bash continuation backslash.

The root `.mcp.json` provides the anonymous configuration for compatible clients. Account access uses an optional static bearer header; OAuth is not required by this server.

## Public tools (no authentication required)

| Tool | Input | Result and limits |
|---|---|---|
| `scan_domain` | Bare `domain`; optional `dkim_selector` | Security score, findings, fix hints, and an opaque report link. Creates a temporary report, consumes scan quota, and may send administrative notifications. |
| `reverse_ip_lookup` | `query`: public IPv4 address or bare domain resolved to IPv4; optional `deep` | Associated domains, network/ASN information and PTR records. Uses the service's index, reverse DNS and public discovery sources. Deep discovery runs automatically when needed. Anonymous results list up to 20 domains. |
| `find_subdomains` | Bare `domain` | Subdomains found in public certificate transparency records (crt.sh / crt.name). Only certificate-listed names are included; this is not a complete inventory. Results may be cached for 7 days. |
| `get_monitoring_signup_link` | Bare `domain` | Registration link with the domain pre-filled, when the user asks about monitoring or alerts. **Does not activate monitoring**, create an account, send email, or consume scan quota. The user must continue on the website to set up monitoring. |

Public lookups do not create scan reports or consume the free scan quota. They query external public sources, cache results and count requests for abuse control. Sharing an IP address does not establish shared ownership or malicious activity, and a missing result does not establish that no associated domains or subdomains exist.

### What `scan_domain` returns

A real scan of a bare domain (`example.com`), including:

SSL expiry · SPF · DKIM · DMARC · MTA-STS · HTTPS redirect · HSTS · CSP · DNSSEC · CAA · BIMI · domain expiration (RDAP) · DNS blocklists · uptime and response time

Plus a 0-100 score, per-check findings with fix hints, and an opaque report link with a server-advertised **7-day expiry**. Anyone holding the link can access it until expiry; do not publish it publicly. The score summarizes the checks performed and is not a guarantee that a domain is secure. See [SECURITY.md](./SECURITY.md) for verification details.

### Example requests

- "Scan example.com and explain the failing security checks."
- "Which domains are associated with this public IPv4 address?"
- "Find subdomains of example.com in public certificate records."
- "Give me a signup link to set up monitoring for example.com."

## Authenticated tools (API key)

| Tool | Purpose |
|---|---|
| `list_domains` | Domains in your account |
| `get_domain_report` | Latest report for one of your domains |
| `scan_my_domain` | Fresh scan of one of your domains (plan limits apply) |
| `get_uptime` | Uptime / latency windows |
| `list_incidents` | Recent downtime / degraded events |

Create a key in the Domain Scan dashboard. The service documents one-time display, hashed storage and revocation of keys; these backend controls are outside this configuration-only repository.

## Documented service limits

| Caller | Advertised limit |
|---|---|
| Anonymous `scan_domain` (IP) | 20 scans / 10 minutes |
| Anonymous `reverse_ip_lookup` / `find_subdomains` (IP) | 20 lookups / 10 minutes; separate from scan quota |
| Authenticated free plan | 30 calls / day; lists up to 50 |
| Authenticated trial | 200 calls / day; lists up to 150 |
| Authenticated Pro | Up to 1,000 calls / hour; complete available lists |

These are the limits advertised by the live MCP instructions on 2026-10-31, not independently verified enforcement guarantees. Anonymous reverse-IP results list up to 20 domains; authenticated list sizes depend on the plan. A complete available list is still limited to the service's sources, not all domains on the internet. Follow the current tool descriptions and server rate-limit errors if the service changes.

The service documents SSRF guards that reject private/loopback IPs and names resolving to private ranges. This repository does not implement these controls. See [SECURITY.md](./SECURITY.md) for the verification scope and outstanding checks.

## Data sent to the service

Calls send the supplied domain, IPv4 address, optional DKIM selector or lookup options to the remote MCP endpoint. Account calls also send the configured bearer key and tool arguments. The service receives normal request metadata, and account queries record API usage and update the key's last-used timestamp. Scans persist temporary reports, consume quota and may send irreversible administrative notifications. Public lookups query external public sources and cache/count requests; they do not create scan reports. The signup-link tool returns a link without activating monitoring or sending email.

Tools do not require passwords or personal information in their arguments; this is not a claim that the service processes no personal data or keeps no logs. Do not include secrets or personal information in tool arguments. Website transactional email and any user-configured email notifications are handled by the service, not by the signup-link tool. See the product's privacy policy for provider and processing details.

Grok Build receives text results; this plugin does not depend on a ChatGPT-specific widget.

## Plugin contents

```
.grok-plugin/plugin.json   # marketplace manifest
.mcp.json                  # remote HTTP MCP only
skills/domain-scan/        # when-to-use guidance for the agent
assets/logo.png            # official Domain Scan logo
SECURITY.md                # threat model
LICENSE                    # MIT
```

No `hooks/`, no `commands/` that shell out, no vendored MCP binary.

## Links

- Product: https://domainscan.tools
- llms.txt: https://domainscan.tools/llms.txt
- Contact: hello@domainscan.tools
- Plugin issues: https://github.com/EnriqueITE/domainscan-mcp-grok-plugin/issues
- MCP endpoint: https://api.domainscan.tools/functions/v1/mcp-server

## License

MIT © CYBERACE LTD
