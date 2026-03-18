---
sidebar_position: 1
---

# CALL_MCP_TOOL

Call a tool on an MCP server via the Hub.

## At a glance
- **Category** Service
- **Version:** 1.0.0
- **Applications:** all
- **Scope:** all
- **Default Service:** HUB

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `server` | The name of the MCP server. | None |true| false |None|None|
| `tool` | The name of the tool to call. | None |true| false |None|None|
| `arguments` | The arguments to pass to the tool. | None |false| false |None|None|

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `object` | The result of the tool call. | false |

## Examples

### Call an MCP tool
```yaml
- step: CALL_MCP_TOOL
  server: filesystem
  tool: read_file
  arguments:
    path: /tmp/test.txt
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
