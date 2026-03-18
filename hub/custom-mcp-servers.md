---
id: custom-mcp-servers
title: Custom MCP Servers
sidebar_label: Custom MCP Servers
sidebar_position: 7
---

# Custom MCP Servers

ProActions Hub allows you to register and run custom MCP (Model Context Protocol) servers. This enables you to extend the Hub's capabilities with specialized tools, data sources, or workflows written in Node.js, Python, or other languages.

## Prerequisites

To run custom scripts, you must make them available to the ProActions Hub container. The recommended approach is to mount a host directory containing your scripts into the container at runtime.

For example, to mount a local directory `./local/scripts` to `/usr/src/app/custom-scripts` in the container:

```bash
-v ./local/scripts:/usr/src/app/custom-scripts:ro
```

## Node.js Example

You can run Node.js-based MCP servers using the `node` executable or `npx`.

### Creating a Simple Server

Here is an example of a simple `weather-server.js` using the `@modelcontextprotocol/sdk`.

**File: `weather-server.js`**

```javascript
#!/usr/bin/env node
import { Server } from '@modelcontextprotocol/sdk/server/index.js';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';
import { CallToolRequestSchema, ListToolsRequestSchema } from '@modelcontextprotocol/sdk/types.js';

const server = new Server(
  {
    name: 'weather-server',
    version: '1.0.0',
  },
  {
    capabilities: {
      tools: {},
    },
  },
);

server.setRequestHandler(ListToolsRequestSchema, async () => {
  return {
    tools: [
      {
        name: 'get_weather',
        description: 'Get current weather for a location',
        inputSchema: {
          type: 'object',
          properties: {
            location: { type: 'string' },
          },
          required: ['location'],
        },
      },
    ],
  };
});

server.setRequestHandler(CallToolRequestSchema, async (request) => {
  if (request.params.name === 'get_weather') {
    const { location } = request.params.arguments;
    return {
      content: [{ type: 'text', text: `Sunny in ${location}` }],
    };
  }
  throw new Error('Tool not found');
});

async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
}

main().catch(console.error);
```

### Configuration

Add the server to your `config.yaml`.

**Using `node` for local scripts:**

```yaml
mcp:
  servers:
    - name: weather
      command: node
      args:
        - /usr/src/app/custom-scripts/weather-server.js
```

**Using `npx` for published packages:**

```yaml
mcp:
  servers:
    - name: sqlite
      command: npx
      args:
        - -y
        - '@modelcontextprotocol/server-sqlite'
        - '--file'
        - '/usr/src/app/custom-scripts/my-db.sqlite'
```

## Python Example

For Python servers, we recommend using `uv` to manage dependencies and execution. This allows you to define dependencies directly in the script.

### Creating a Simple Server

Here is an example of an `echo.py` server using `fastmcp`.

**File: `echo.py`**

```python
# /// script
# dependencies = [
#   "mcp[cli]"
# ]
# ///

from mcp.server.fastmcp import FastMCP

mcp = FastMCP("Echo")

@mcp.tool()
def echo(message: str) -> str:
    """Echo back the message"""
    return f"Echo: {message}"

if __name__ == "__main__":
    mcp.run()
```

### Configuration

Configure the server to use `uv` in `config.yaml`:

```yaml
mcp:
  servers:
    - name: echo
      command: uv
      args:
        - run
        - /usr/src/app/custom-scripts/echo.py
```

## Deployment

To deploy with custom servers, run the container with the volume mount:

**Docker:**

```bash
docker run -d \
  -p 3000:3000 \
  -v ./config/config.yaml:/usr/src/app/dist/config.yaml \
  -v ./my-local-scripts:/usr/src/app/custom-scripts:ro \
  artifactory.eidosmedia.sh/docker-releases/proactions/proactions-hub:latest
```

**Podman:**

```bash
podman run -d \
  -p 3000:3000 \
  -v ./config/config.yaml:/usr/src/app/dist/config.yaml \
  -v ./my-local-scripts:/usr/src/app/custom-scripts:ro \
  artifactory.eidosmedia.sh/docker-releases/proactions/proactions-hub:latest
```
