---
sidebar_position: 1
---

# SHOW_PROGRESS

Show a progress bar with optional status text. Use `position` to place it and `autoHide` to auto-hide after completion.

## At a glance
- **Category** UI
- **Version:** 1.0.8
- **Applications:** all
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `status` | Status text to display next to the progress bar. Supports variable resolution. | None |false| true |None|None|
| `position` | Position of the progress bar. Common values: 'bottom', 'top'. | None |false| false |None|None|
| `autoHide` | Whether the progress bar should auto-hide when completed. | None |false| false |None|None|

## Examples

### Show progress at bottom
```yaml
- step: SHOW_PROGRESS
  status: "Processing..."
  position: "bottom"
  autoHide: true
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
