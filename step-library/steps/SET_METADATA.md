---
sidebar_position: 1
---

# SET_METADATA

Set a field value in the object panel using a selector or XPath. Supports tagsinput and plain fields.

## At a glance
- **Category** Editor
- **Aliases** CTX_SET_METADATA
- **Version:** 1.0.0
- **Applications:** all
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `selector` | CSS selector of the input element to update. | None |true| false |None|None|

## Inputs
| Type | Description | Default | Required | Resolved |
|---|---|:---:|:---:|:---:|
| `text` | Text value to set in the field (used for non-tagsinput fields and fallback) | None | false | false |
| `list` | List of values to set (used for tagsinput fields) | None | false | false |

## Examples

### Set a text field
```yaml
  - step: SET
    text: "The value"
  - step: SET_METADATA
   selector: "#metadata-title"
 ```

### Set tagsinput field from list input
```yaml
 - step: SET
   list:
     - entry1
     - entry2
 - step: SET_METADATA
   selector: "#tags"
 ```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
