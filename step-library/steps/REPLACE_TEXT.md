---
sidebar_position: 1
---

# REPLACE_TEXT

Replace existing text at the given location (cursor paragraph, cursor or xpath) with the provided content. Content can come from a flow variable (using `in`) or the default text input.

## At a glance
- **Category** Editor
- **Aliases** CTX_REPLACE_TEXT
- **Version:** 1.0.0
- **Applications:** all
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `in` | Optional text or variable name to read the content from the flowContext. If omitted, the step uses the default text input. | None |false| true |None|None|
| `at` | Where to replace the text. Options: XPATH, CURSOR_PARAGRAPH, CURSOR. | None |false| false |None|None|
| `xpath` | XPath expression used when `at` is XPATH. | None |Conditional| false |None|Required when `at` = `XPATH`|
| `forceWrite` | Write content even if the document is considered read-only. | None |false| false |None|None|

## Examples

### Replace text at cursor paragraph
```yaml
- step: SET
  text: "Replacement text"

- step: REPLACE_TEXT
  at: CURSOR_PARAGRAPH
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
