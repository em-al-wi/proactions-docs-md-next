---
sidebar_position: 1
---

# INSERT_TEXT

Insert text at the given location (cursor, xpath or report). If the step reads text from the flowContext, you can use a preceding SET step to provide the content.

## At a glance
- **Category** Editor
- **Aliases** CTX_INSERT_TEXT
- **Version:** 1.0.0
- **Applications:** all
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `in` | Optional path or variable name to read the content from the flowContext. If omitted, the step uses the content from the default text input. | None |false| true |None|None|
| `at` | Where to insert the text. Use CURSOR (default), XPATH, or REPORT. | None |false| false |None|None|
| `xpath` | XPath expression used when `at` is XPATH. | None |Conditional| false |None|Required when `at` = `XPATH`|
| `forceWrite` | Write content even if the document is considered read-only. | None |false| false |None|None|

## Inputs
| Type | Description | Default | Required | Resolved |
|---|---|:---:|:---:|:---:|
| `text` | Text content to insert. If omitted, reads from the `in` parameter or default text input. | None | false | false |

## Examples

### Insert text from a SET step
```yaml
- step: SET
  text: "This text will be inserted"
- step: INSERT_TEXT
  at: CURSOR
```

### Insert text from a flow variable
```yaml
- step: INSERT_TEXT
  in: "{{ flowContext.myText }}"
  at: XPATH
  xpath: "/doc/story/header/title/p"
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
