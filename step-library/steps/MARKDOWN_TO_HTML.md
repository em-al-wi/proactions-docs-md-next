---
sidebar_position: 1
---

# MARKDOWN_TO_HTML

Converts markdown text to HTML and sets it as text output.

## At a glance
- **Category** Utils
- **Version:** 1.0.10
- **Applications:** all
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `text` | Text input or flow variable to convert. If omitted, the default text input is used. | None |false| true |None|None|

## Examples

### Convert markdown to HTML
```yaml
- step: MARKDOWN_TO_HTML
  text: "# Hello\\nThis is **markdown**"
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
