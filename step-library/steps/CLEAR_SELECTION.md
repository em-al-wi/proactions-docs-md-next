---
sidebar_position: 1
---

# CLEAR_SELECTION

Clears the current text selection and optionally repositions the cursor to the start or end of the selection. Available in Swing editor context only.

## At a glance
- **Category** Editor
- **Version:** 1.0.7
- **Applications:** Swing
- **Scope:** story, report, dwp, dashboard

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `cursorPosition` | Where to move the cursor after clearing selection. Use "anchorStart" to move to the start or "anchorEnd" to move to the end. | None |false| false |None|None|

## Examples

### Clear selection and move cursor to end
```yaml
- step: CLEAR_SELECTION
  cursorPosition: "anchorEnd"
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
