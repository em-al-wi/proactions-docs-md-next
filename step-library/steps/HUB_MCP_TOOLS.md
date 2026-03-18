---
sidebar_position: 1
---

# HUB_MCP_TOOLS

List available tools from an MCP server via the Hub.

## At a glance
- **Category** Service
- **Version:** 1.2.0
- **Applications:** all
- **Scope:** all
- **Default Service:** HUB

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `server` | The name of the MCP server to list tools from. | None |false| false |None|None|

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `object` | The list of available tools (and resources/prompts if provided by the Hub API). | false |

## Examples

### List all MCP servers and tools
```yaml
- step: HUB_MCP_TOOLS
```

### List MCP tools
```yaml
- step: HUB_MCP_TOOLS
  server: search-server
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
