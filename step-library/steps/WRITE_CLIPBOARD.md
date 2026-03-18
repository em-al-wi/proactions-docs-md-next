---
sidebar_position: 1
---

# WRITE_CLIPBOARD

Writes text to the user clipboard. Reads the content from the configured input (cfg.text) or from the default text input in the flow context.

## At a glance
- **Category** Browser
- **Version:** 1.0.3
- **Applications:** all
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `text` | Text to write to clipboard. If omitted, uses the default text input from flow context. | None |false| true |None|None|

## Examples

### Write text from flowContext (use SET before to provide text)
```yaml
- step: SET
  text: "Some text to copy"

- step: WRITE_CLIPBOARD
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
