---
sidebar_position: 1
---

# MONITOR_TABLE

Display tabular structured data in the Flow Monitor.

## At a glance
- **Category** UI
- **Version:** 1.0.0
- **Applications:** all
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `data` | Array of objects to render as a table. | None |true| true |None|None|
| `title` | Optional title displayed above the table. | None |false| true |None|None|
| `maxRows` | Maximum number of rows rendered before truncation. | None |false| false |None|None|

## Examples

### Render table data
```yaml
- step: MONITOR_TABLE
  title: "Summary"
  data: "{{flowContext.list}}"
  maxRows: 20
  ```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
