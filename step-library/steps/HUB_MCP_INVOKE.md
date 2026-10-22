---
sidebar_position: 1
---

# HUB_MCP_INVOKE

Invoke a tool on an MCP server via the Hub.

## At a glance
- **Category** Service
- **Version:** 1.2.0
- **Applications:** all
- **Scope:** all
- **Default Service:** HUB

## When to Use

Use **HUB_MCP_INVOKE** to execute a named tool on a Hub-connected MCP server with structured arguments.

## Data Flow

Reads resolved `server`, `tool`, and `arguments` values. Writes the invocation result to `object` or its configured output name.

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `server` | The name of the MCP server. | None |true| false |None|None|
| `tool` | The name of the tool to invoke. | None |true| false |None|None|
| `arguments` | The arguments to pass to the tool. | None |false| false |None|None|

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `object` | The result of the tool invocation. | false |

## Examples

### Invoke an MCP tool
```yaml
- step: HUB_MCP_INVOKE
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
