---
sidebar_position: 1
---

# MONITOR_JSON

Display JSON structured data in the Flow Monitor.

## At a glance
- **Category** UI
- **Version:** 1.0.0
- **Applications:** all
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `data` | Object or array data to render as JSON. | None |true| true |None|None|
| `title` | Optional title displayed above JSON. | None |false| true |None|None|
| `maxDepth` | Maximum depth for JSON rendering. | None |false| false |None|None|

## Examples

### Render object as JSON
```yaml
- step: MONITOR_JSON
  title: "Model response"
  data: "{{flowContext.object}}"
  maxDepth: 4
  ```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
