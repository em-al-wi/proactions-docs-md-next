---
sidebar_position: 1
---

# SAVE_DOCUMENT

Saves the current document. Optionally closes the document after saving.

## At a glance
- **Category** Editor
- **Version:** 1.2.0
- **Applications:** all
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `close` | Whether to close the document after saving. Supports variable resolution. | false |false| true |None|None|

## Examples

### Save the document
```yaml
- step: SAVE_DOCUMENT
```

### Save and close the document
```yaml
- step: SAVE_DOCUMENT
  close: true
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
