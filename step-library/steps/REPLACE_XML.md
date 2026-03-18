---
sidebar_position: 1
---

# REPLACE_XML

Replace XML content at the given location. Content can be sourced from a flow variable (`in`) or the default text input.

## At a glance
- **Category** Editor
- **Aliases** CTX_REPLACE_XML
- **Version:** 1.0.0
- **Applications:** all
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `in` | Optional text or variable name to read the XML content from the flowContext. | None |false| true |None|None|
| `at` | Where to replace the XML. Options: XPATH, CURSOR_PARAGRAPH, CURSOR. | None |false| false |None|None|
| `xpath` | XPath expression used when `at` is XPATH. | None |Conditional| false |None|Required when `at` = `XPATH`|
| `forceWrite` | Write content even if the document is considered read-only. | None |false| false |None|None|

## Examples

### Replace XML at XPath
```yaml
- step: SET
  text: "<section>Updated</section>"

- step: REPLACE_XML
  at: XPATH
  xpath: "/doc/story/grouphead/headline/p"
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
