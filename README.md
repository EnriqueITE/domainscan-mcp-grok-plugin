<p align="center">
  <a href="https://domainscan.tools">
    <img src="assets/logo.png" alt="Domain Scan by CYBERACE" width="160">
  </a>
</p>

# Domain Scan: Grok Build plugin

Official thin plugin for **[Domain Scan by CYBERACE](https://domainscan.tools)**.

Scan domain security from Grok Build. Visit **[domainscan.tools](https://domainscan.tools)** for the web scanner and monitoring dashboard.

This package **only configures a remote HTTPS MCP server**. It does not install local binaries, hooks, or scripts. Grok talks to:

`https://api.domainscan.tools/functions/v1/mcp-server`

## Ownership and maintenance

Domain Scan is a CYBERACE LTD product. This official plugin is published and maintained by Enrique, CEO of CYBERACE LTD, through his personal GitHub account [EnriqueITE](https://github.com/EnriqueITE).

The source remains at [EnriqueITE/domainscan-mcp-grok-plugin](https://github.com/EnriqueITE/domainscan-mcp-grok-plugin). Questions about ownership or maintenance can be sent to hello@domainscan.tools.

This package targets **Grok Build**. Listing it in the xAI plugin marketplace does not establish availability in the general Grok chat or X.

## Security posture (read this)

- **No local code execution** from this plugin; see [SECURITY.md](./SECURITY.md).
- **Anonymous** use exposes only `scan_domain` (public DNS/TLS/HTTP signals).
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
# Anonymous free scans
grok mcp add --transport http domain-scan https://api.domainscan.tools/functions/v1/mcp-server

# Optional: your account tools
grok mcp add --transport http domain-scan https://api.domainscan.tools/functions/v1/mcp-server \
  --header 'Authorization: Bearer ${DOMAINSCAN_API_KEY}'
```

The commands above use Bash syntax. The single quotes preserve the environment-variable reference for Grok to expand when it loads the configuration. Set `DOMAINSCAN_API_KEY` in your local environment; do not paste the key into chat or commit it. In PowerShell, write the authenticated command on one line without the Bash continuation backslash.

The root `.mcp.json` provides the anonymous configuration for compatible clients. Account access uses an optional static bearer header; OAuth is not required by this server.

## What `scan_domain` returns

A real scan of a bare domain (`example.com`), including:

SSL expiry · SPF · DKIM · DMARC · MTA-STS · HTTPS redirect · HSTS · CSP · DNSSEC · CAA · BIMI · domain expiration (RDAP) · DNS blocklists

Plus a 0-100 score, per-check findings with fix hints, and an opaque report link with a server-advertised 48-hour expiry. The score summarizes the checks performed and is not a guarantee that a domain is secure. See [SECURITY.md](./SECURITY.md) for verification details.

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
| Anonymous (IP) | 20 scans / 10 minutes (shared with website free scan) |
| Authenticated (API key) | 100 calls / hour |

The service documents SSRF guards that reject private/loopback IPs and names resolving to private ranges. This repository does not implement these controls. See [SECURITY.md](./SECURITY.md) for the verification scope and outstanding checks.

## Data sent to the service

Scanning sends the domain and optional DKIM selector to the remote MCP endpoint. Account calls also send the configured bearer key and any tool arguments. The service creates a temporary scan report and receives normal network request metadata. Tools do not require passwords or personal information in their arguments; this is not a claim that the service processes no personal data or keeps no logs. Do not include secrets or personal information in tool arguments.

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
