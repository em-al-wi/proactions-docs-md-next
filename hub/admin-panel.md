---
id: admin-panel
title: Admin Panel
sidebar_label: Admin Panel
sidebar_position: 10
---

# Admin Panel

ProActions Hub includes a built-in admin panel for monitoring runtime metrics, reviewing configuration, and diagnosing issues — all from a browser-based dashboard. The panel is protected by stateless API key authentication independent of the main auth system.

## Quick Start

### 1. Enable Admin Authentication

```yaml
auth:
  admin:
    enabled: true
    apiKeys:
      - ${PROACTIONS_ADMIN_KEY}
```

Set the environment variable (minimum 32 characters):

```bash
PROACTIONS_ADMIN_KEY=$(openssl rand -base64 32)
```

### 2. Access the Panel

Open your browser and navigate to:

```
https://swing.your-env.em-cms.cloud/swing/proactions/admin/panel
```

Enter your admin key in the login screen and click **Sign In**.

:::warning Key Length Requirement
Admin API keys must be at least 32 characters long. Shorter keys are rejected during configuration validation.
:::

## Configuration

### Admin Auth Settings

```yaml
auth:
  admin:
    enabled: true
    apiKeys:
      - ${PROACTIONS_ADMIN_KEY_1}
      - ${PROACTIONS_ADMIN_KEY_2}
    allowedNetworks:
      - 127.0.0.1
      - 10.0.0.0/8
```

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `enabled` | boolean | `false` | Enable admin authentication |
| `apiKeys` | string[] | `[]` | API keys for admin access (min 32 chars each, required when enabled) |
| `allowedNetworks` | string[] | `[]` | Optional IP/CIDR allowlist (empty = all IPs allowed) |

### Multiple API Keys

You can configure multiple admin keys to support key rotation or multiple administrators:

```yaml
auth:
  admin:
    enabled: true
    apiKeys:
      - ${PROACTIONS_ADMIN_KEY_PRIMARY}
      - ${PROACTIONS_ADMIN_KEY_SECONDARY}
```

### Network Restrictions

Restrict admin access to specific IP addresses or CIDR ranges:

```yaml
auth:
  admin:
    allowedNetworks:
      - 127.0.0.1           # Localhost only
      - 10.0.0.0/8          # Private network range
      - 192.168.1.50        # Specific workstation
```

When `allowedNetworks` is configured, requests from non-listed IPs receive a `403 Forbidden` response.

:::tip Reverse Proxy
If ProActions Hub is behind a reverse proxy, enable `hub.security.trustProxy` so the guard uses the real client IP from `X-Forwarded-For` instead of the proxy IP.
:::

### Independence from Main Auth

Admin authentication is fully independent of the main authentication system (EDAPI or token-based). This means:

- Admin panel works even when `auth.enabled: false`
- Admin keys are separate from EDAPI credentials or auth tokens
- No session or cookie is required — every request sends the key via the `x-admin-key` header

```yaml
# Main auth disabled, admin auth enabled
auth:
  enabled: false
  admin:
    enabled: true
    apiKeys:
      - ${PROACTIONS_ADMIN_KEY}
```

## Dashboard Tabs

The admin panel organizes information into six tabs.

### Dashboard

Overview of the running instance:

- **Process information** — PID, hostname, Node.js version, uptime, platform
- **Memory usage** — RSS, heap used, heap total, external
- **Configuration summary** — counts for configured targets, proxies, tools, and MCP servers
- **Runtime usage charts** — visual breakdown of request distribution across providers and proxies

### AI-Link

AI provider usage metrics collected in real time:

- **Summary cards** — total requests, token usage, streaming ratio, error rate, active models
- **Token usage by provider** — horizontal stacked bar chart (prompt vs completion tokens)
- **Latency by model** — average response time with min/max range
- **Finish reason distribution** — doughnut chart showing completion, stop, length, etc.
- **Detailed tables** — per-provider, per-model, and per-action breakdowns with tokens, errors, and latency
- **Configured targets** — table showing all AI targets from configuration

When no AI-Link requests have been made yet, a placeholder card is shown.

### Proxies

Proxy configuration and usage:

- **Configured proxies** — ID, URL, and header removal settings
- **Usage statistics** — request counts per proxy target

### MCP

MCP (Model Context Protocol) server status and metrics:

- **Server health** — connection status, transport type, auto-connect setting
- **Server metrics** — average latency (with min/max), error count and error rate
- **Tool inventory** — available tools per server with invocation counts
- **Tool-level metrics** — per-tool latency and error badges

### Config

Full runtime configuration view with sensitive values automatically redacted. Displayed as formatted JSON.

### System

Process-level information:

- **Node.js version**, platform, PID
- **Uptime** — formatted as days/hours/minutes
- **Memory breakdown** — detailed heap and RSS statistics
- **Runtime usage snapshot** — raw usage data

## Toolbar

The navigation bar provides quick actions:

| Control | Description |
|---------|-------------|
| **Auto Refresh** | Toggle automatic data refresh every 30 seconds |
| **Refresh** | Manually reload all data |
| **Reset Statistics** | Clear all runtime metrics (with confirmation dialog) |
| **Theme Toggle** | Switch between light and dark mode (persisted in localStorage) |
| **Logout** | Clear stored admin key and return to login screen |

## API Endpoints

All endpoints except `/admin/panel` require the `x-admin-key` header.

### Panel

```
GET /admin/panel
```

Serves the admin panel HTML page. This endpoint is public (no authentication required) — authentication happens client-side via the API key.

### Runtime Diagnostics

```
GET /admin/runtime
```

Returns process information, configuration summary, runtime usage, and MCP connection health. This is the primary endpoint used by the dashboard on every refresh.

**Response structure:**

```json
{
  "process": {
    "pid": 1234,
    "hostname": "hub-server",
    "nodeVersion": "v22.x.x",
    "uptimeSeconds": 86400,
    "platform": "linux",
    "memoryMb": { "rss": 120, "heapUsed": 80, "heapTotal": 100, "external": 5 }
  },
  "configurationSummary": {
    "targets": { "count": 5, "items": [...] },
    "proxies": { "count": 2, "items": [...] },
    "tools": { "count": 1, "items": [...] },
    "mcp": { "count": 3, "items": [...] }
  },
  "runtimeUsage": { ... },
  "mcpConnections": [ ... ]
}
```

### Runtime Usage

```
GET /admin/runtime/usage
```

Returns only the runtime usage statistics (subset of the diagnostics endpoint).

### Reset Statistics

```
POST /admin/runtime/reset
```

Clears all runtime usage counters and resets the uptime anchor. This action is logged at `warn` level with the admin username and client IP.

**Response:**

```json
{
  "success": true,
  "message": "Runtime statistics have been reset"
}
```

:::info Audit Trail
Every statistics reset is logged with the admin identity and client IP address for audit purposes.
:::

### Configuration Endpoints

| Endpoint | Description |
|----------|-------------|
| `GET /admin/config` | Full configuration (all sections, secrets redacted) |
| `GET /admin/config/targets` | AI target configurations |
| `GET /admin/config/proxies` | Proxy configurations |
| `GET /admin/config/mcp` | MCP server configurations |
| `GET /admin/config/auth` | Authentication configuration |
| `GET /admin/config/hub` | Hub/server configuration |

All configuration responses are sanitized — sensitive fields such as API keys, tokens, passwords, and secrets are automatically redacted.

## Security

### Authentication Mechanism

- **Header-based**: Every API request includes the `x-admin-key` header
- **Timing-safe comparison**: Key validation uses `crypto.timingSafeEqual` to prevent timing attacks
- **Stateless**: No sessions or cookies — the key is validated on every request
- **Header cleanup**: The admin key header is removed from the request after validation

### Error Responses

| Status | Meaning |
|--------|---------|
| 401 | Missing or invalid admin key |
| 403 | Admin access not configured, or client IP not in `allowedNetworks` |

### Sensitive Data Redaction

All configuration and diagnostic responses pass through the sensitive data filter. Fields matching keywords like `password`, `token`, `apiKey`, `secret`, `authorization`, and `key` are replaced with `[REDACTED]`.

### Recommendations

1. **Use strong keys** — generate with `openssl rand -base64 32` or equivalent
2. **Restrict networks** — use `allowedNetworks` to limit access to known admin IPs
3. **Enable trust proxy** — when behind a reverse proxy, so IP allowlisting works correctly
4. **Rotate keys** — configure multiple keys and rotate periodically
5. **Monitor resets** — statistics reset actions are logged for audit compliance

## Troubleshooting

### Panel Loads but Login Fails

- Verify your admin key is at least 32 characters
- Check that `auth.admin.enabled` is `true` in configuration
- If using `allowedNetworks`, confirm your IP is listed (check behind proxy)

### Panel Shows "Failed to Fetch Data"

- The admin key may have expired or been rotated — try logging out and back in
- Check that the Hub process is running and healthy
- Review browser developer console for network errors

### No AI-Link Metrics Shown

- AI-Link metrics are collected in memory at runtime — they start empty after each restart
- Make some AI-Link requests first, then refresh the panel
- Verify that AI-Link targets are configured and working

## Next Steps

- [Security Hardening](./operations/security-hardening.md) for production deployment best practices
- [Logging Configuration](./config/logging.md) for monitoring and audit compliance
- [Operations Guide](./operations/operations-guide.md) for maintenance procedures
