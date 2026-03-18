---
sidebar_position: 1
---

# INSERT_XML

Insert XML content at the given location. The content can be provided via the flowContext (using `in`) or via the default text input.

## At a glance
- **Category** Editor
- **Aliases** CTX_INSERT_XML
- **Version:** 1.0.0
- **Applications:** all
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `in` | Optional path or variable name to read the XML content from the flowContext. If omitted, the step uses the default text input. | None |false| true |None|None|
| `at` | Where to insert the XML. Use CURSOR (default) or XPATH. | None |false| false |None|None|
| `xpath` | XPath expression used when `at` is XPATH. | None |Conditional| false |None|Required when `at` = `XPATH`|
| `position` | Position mode for insertion when using cursor (e.g. insertInline, insertBefore, insertAfter). | insertInline |false| false |None|None|
| `forceWrite` | Write content even if the document is considered read-only. | None |false| false |None|None|

## Examples

### Insert XML at cursor
```yaml
- step: SET
  text: "<p>Inserted paragraph</p>"

- step: INSERT_XML
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
