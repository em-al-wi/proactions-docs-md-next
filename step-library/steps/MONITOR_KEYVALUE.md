---
sidebar_position: 1
---

# MONITOR_KEYVALUE

Display key-value structured data in the Flow Monitor.

## At a glance
- **Category** UI
- **Version:** 1.0.0
- **Applications:** all
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `data` | Object to render as key-value pairs. | None |true| true |None|None|
| `title` | Optional title displayed above key-value pairs. | None |false| true |None|None|

## Examples

### Render key-value summary
```yaml
- step: MONITOR_KEYVALUE
  title: "Result summary"
  data: "{{flowContext.object}}"
  ```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
