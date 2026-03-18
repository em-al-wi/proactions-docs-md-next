---
id: mcp-gateway
title: MCP Gateway
sidebar_label: MCP Gateway
sidebar_position: 6
---

# MCP Gateway

The **MCP Gateway** allows ProActions Hub to host and connect to [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) servers. This enables the ProActions Browser Client (Swing/Prime) to access backend tools, filesystems, databases, and other resources through a standardized protocol.

:::danger Third-Party Software Disclaimer
Enabling the MCP Gateway allows the dynamic installation and execution of third-party software packages (via `npx`, `uv`, `pip`, etc.) from external sources such as npm and PyPI.

**Important:**

- These packages are **not** provided, maintained, or audited by Eidosmedia.
- Eidosmedia assumes **no responsibility** for the security, stability, or licensing of any third-party MCP servers or tools you choose to run.
- You are solely responsible for vetting and trusting the MCP servers you configure.
- Ensure your environment has appropriate network security controls (firewalls, egress filtering) to restrict what these tools can access.
  :::

## Features

- **Multi-Transport Support**: Connects to MCP servers via `stdio`, `http`, or `websocket` transports.
- **Remote Server Support**: Connect to remote MCP servers over HTTP (Streamable HTTP) or WebSocket.
- **Dynamic Execution**: Spawns local MCP servers on demand using `npx`, `python`, `uv`, or other allowed commands.
- **Security**: Strict command allowlist and path traversal protection for local servers.
- **Lifecycle Management**: Auto-connect/disconnect, graceful shutdown, connection timeouts, and cleanup.
- **Monitoring**: Health and statistics tracking for all connected servers.

## Configuration

MCP servers are configured in the `config.yaml` file under the `mcp` section.

### Transport Types

The MCP Gateway supports three transport types for connecting to MCP servers:

| Transport   | Description                                                                 | Use Case                                    |
| ----------- | --------------------------------------------------------------------------- | ------------------------------------------- |
| `stdio`     | Spawns a local subprocess and communicates via stdin/stdout (default)       | Local tools, npm/Python packages            |
| `http`      | Connects to a remote server via Streamable HTTP (POST for requests, SSE for responses) | Remote MCP servers with HTTP endpoints |
| `websocket` | Connects to a remote server via WebSocket                                   | Real-time remote MCP servers                |

### Local Server Configuration (stdio)

Use `stdio` transport (the default) to spawn local MCP servers as subprocesses.

```yaml
mcp:
  servers:
    - name: filesystem
      transport: stdio  # Optional, stdio is the default
      command: npx
      args: ['-y', '@modelcontextprotocol/server-filesystem', '/tmp']
      autoConnect: false
      env:
        NODE_ENV: production
```

### Remote Server Configuration (HTTP)

Use `http` transport to connect to remote MCP servers via Streamable HTTP. This is ideal for centralized MCP servers or third-party hosted services.

```yaml
mcp:
  servers:
    - name: remote-tools
      transport: http
      url: https://mcp.example.com/v1
      headers:
        Authorization: "Bearer ${REMOTE_MCP_TOKEN}"
      autoConnect: true
      connectTimeout: 15000
      toolTimeout: 30000
```

:::tip Authentication
Use the `headers` option to pass authentication tokens or API keys to remote HTTP servers. Environment variable substitution is supported.
:::

### Remote Server Configuration (WebSocket)

Use `websocket` transport to connect to remote MCP servers via WebSocket for low-latency, bidirectional communication.

```yaml
mcp:
  servers:
    - name: ws-tools
      transport: websocket
      url: wss://mcp.example.com/ws
      autoConnect: true
```

### Supported Runtimes (stdio only)

The Hub container supports both Node.js and Python-based MCP servers.

#### Node.js (via npx)

Use `npx` to run Node.js-based servers without manual installation.

```yaml
mcp:
  servers:
    - name: filesystem
      command: npx
      args: ['-y', '@modelcontextprotocol/server-filesystem', '/allowed/path']
```

#### Python (via uv)

The container includes [uv](https://github.com/astral-sh/uv), a fast Python package installer and resolver. You can use `uvx` to run Python tools ephemerally.

```yaml
mcp:
  servers:
    - name: python-tool
      command: uvx
      args: ['mcp-server-package', '--arg', 'value']
```

### Configuration Options

#### Common Options (all transports)

| Option           | Type     | Default      | Description                                                          |
| ---------------- | -------- | ------------ | -------------------------------------------------------------------- |
| `name`           | string   | **Required** | Unique identifier for the server. Used in the `x-target` header.     |
| `transport`      | string   | `stdio`      | Transport type: `stdio`, `http`, or `websocket`.                     |
| `autoConnect`    | boolean  | `false`      | If `true`, connects to the server immediately on startup.            |
| `connectTimeout` | number   | `30000`      | Connection timeout in milliseconds.                                  |
| `toolTimeout`    | number   | `60000`      | Tool invocation timeout in milliseconds.                             |
| `allowedTools`   | string[] | `[]`         | If set, only these tools are exposed (allowlist).                    |
| `ignoredTools`   | string[] | `[]`         | If set, these tools are hidden (blocklist).                          |

#### stdio Transport Options

| Option    | Type     | Default      | Description                                                          |
| --------- | -------- | ------------ | -------------------------------------------------------------------- |
| `command` | string   | **Required** | Executable command. Must be in the allowlist.  |
| `args`    | string[] | `[]`         | Arguments passed to the command.                                     |
| `env`     | object   | `{}`         | Environment variables for the subprocess.                            |

#### http Transport Options

| Option    | Type   | Default      | Description                                         |
| --------- | ------ | ------------ | --------------------------------------------------- |
| `url`     | string | **Required** | URL of the remote MCP server (e.g., `https://...`). |
| `headers` | object | `{}`         | Custom HTTP headers (e.g., for authentication).     |

#### websocket Transport Options

| Option | Type   | Default      | Description                                        |
| ------ | ------ | ------------ | -------------------------------------------------- |
| `url`  | string | **Required** | URL of the remote MCP server (e.g., `wss://...`).  |

## Usage

Interact with the MCP Gateway via REST API endpoints. All endpoints require authentication.

### Invoke a Tool

Execute a tool on a specific MCP server.

**Endpoint:** `POST /mcp/invoke`

**Headers:**

- `x-target`: Name of the MCP server (matches `config.yaml`).
- `Authorization`: Bearer token.

**Body:**

```json
{
  "tool": "read_file",
  "arguments": {
    "path": "/tmp/demo.txt"
  }
}
```

**Response:**

```json
{
  "content": [
    {
      "type": "text",
      "text": "File content here..."
    }
  ],
  "isError": false
}
```

### List Tools

Get a list of available tools from all configured servers.

**Endpoint:** `GET /mcp/tools`

**Query Parameters:**

- `server` (optional): Filter by specific server name.

### Get Statistics

Retrieve connection status and health metrics for all servers.

**Endpoint:** `GET /mcp/stats`

## Security

### Command Allowlist (stdio only)

For `stdio` transport, only specific commands are allowed to be executed. The default allowlist includes:

- `node`
- `npx`
- `python`, `python3`
- `uv`, `uvx`

Attempts to use other commands will result in a configuration validation error.

:::info Remote Transports
The command allowlist only applies to `stdio` transport. Remote transports (`http`, `websocket`) do not spawn processes and are not subject to command validation.
:::

### File Access & Permissions (stdio only)

- The Gateway runs as a **non-root user (1001)**.
- **npm/npx**: A writable cache directory (`.npm`) is configured in the container to allow `npx` to fetch packages at runtime.
- **Python/uv**: A writable `HOME` directory is configured to allow `pip` and `uv` to install packages.
- **Path Traversal**: Arguments are validated to prevent directory traversal attacks (e.g., `../`).

### Remote Server Security (http/websocket)

When connecting to remote MCP servers:

1. **Always use TLS**: Use `https://` or `wss://` URLs to encrypt traffic.
2. **Authenticate**: Pass authentication tokens via the `headers` option (HTTP) or query parameters in the URL.
3. **Network Controls**: Restrict outbound connections from the Hub container to only trusted MCP server endpoints.
4. **Validate Trust**: Only connect to MCP servers you trust, as they can execute arbitrary operations.

```yaml
mcp:
  servers:
    - name: secure-remote
      transport: http
      url: https://trusted-mcp.internal.example.com/v1
      headers:
        Authorization: "Bearer ${SECURE_TOKEN}"
        X-API-Key: ${API_KEY}
```

### Best Practices

1. **Least Privilege**: Only map necessary volumes to the container (stdio).
2. **Read-Only**: Mount configuration files as read-only.
3. **Environment Variables**: Use environment variables for sensitive credentials (API keys) instead of hardcoding them.
4. **Timeouts**: Configure appropriate `connectTimeout` and `toolTimeout` values to prevent hanging connections.

```yaml
mcp:
  servers:
    - name: secure-server
      command: npx
      args: ['-y', 'my-server']
      env:
        API_KEY: ${MY_SECRET_KEY}
      connectTimeout: 10000
      toolTimeout: 30000
```
