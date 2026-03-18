---
sidebar_position: 1
---

# CHANGE_VIEW_SIZE

Requests a change of the view port size. Useful to resize the command palette view in Prime 8.

## At a glance
- **Category** UI
- **Version:** 1.0.12
- **Applications:** Prime 8
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `width` | Target width in pixels. Can be a number or an expression resolved against the flowContext. | None |true| true |None|None|
| `height` | Target height in pixels. Can be a number or an expression resolved against the flowContext. | None |true| true |None|None|

## Examples

### Change view size to 800x600
```yaml
- step: CHANGE_VIEW_SIZE
  width: 800
  height: 600
```

### Change view size from flow variables
```yaml
- step: SET
  width: 1024
  height: 768

- step: CHANGE_VIEW_SIZE
  width: "{{flowContext.width}}"
  height: "{{flowContext.height}}"
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
