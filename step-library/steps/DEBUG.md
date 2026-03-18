---
sidebar_position: 1
---

# DEBUG

Logs debug information about the current flow context and step configuration. Optionally triggers a debugger breakpoint.

## At a glance
- **Category** Control
- **Version:** 1.0.6
- **Applications:** all
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `id` | Optional identifier for the debug step, included in the log message. | None |false| false |None|None|
| `omitDebugger` | If true, skips triggering the debugger breakpoint. | false |false| false |None|None|

## Examples

### Basic debug step
```yaml
 - step: DEBUG
   id: "myDebugPoint"
   omitDebugger: false
 ```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
