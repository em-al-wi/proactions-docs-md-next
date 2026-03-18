---
sidebar_position: 1
---

# INSERT_LIST

Insert a list of items into the document. The items can be provided as a list input or be generated from a text input (converted to a list using ProcessToList).

## At a glance
- **Category** Editor
- **Aliases** CTX_INSERT_LIST
- **Version:** 1.0.0
- **Applications:** all
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `in` | Optional text or variable name to read the content (text) from the flowContext. If omitted, the step uses the default text input. | None |false| true |None|None|
| `containerElement` | Container tag to use for the list (default 'ul'). | None |false| false |None|None|
| `listItemElement` | List item tag to use for entries (default 'li'). | None |false| false |None|None|
| `omitContainer` | When true, inserts items one-by-one instead of a full container. | None |false| false |None|None|
| `xpath` | XPath expression used when `at` is XPATH. | None |Conditional| false |None|Required when `at` = `XPATH`|
| `at` | Where to insert the list. Use CURSOR (default) or XPATH. | None |false| false |None|None|
| `position` | Position mode for insertion when using cursor (e.g. insertInline, insertBefore, insertAfter). | insertInline |false| false |None|None|
| `forceWrite` | Write content even if the document is considered read-only. | None |false| false |None|None|

## Inputs
| Type | Description | Default | Required | Resolved |
|---|---|:---:|:---:|:---:|
| `list` | List of items to insert. If omitted, the step converts the text input to a list. | None | false | false |
| `text` | Text content to convert to a list if no list input is provided. | None | false | false |

## Examples

### Insert list from text
```yaml
- step: SET
  text: "Item 1\\nItem 2\\nItem 3"

- step: INSERT_LIST
  containerElement: "ol"
  listItemElement: "li"
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
