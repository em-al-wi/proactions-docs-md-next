---
sidebar_position: 1
---

# GET_TEXT_CONTENT

Retrieves text content from the document or a specific location and sets it as output.

## At a glance
- **Category** Editor
- **Aliases** CTX_GET_TEXT_CONTENT
- **Version:** 1.0.1
- **Applications:** all
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `at` | Location to read text from. Options: CURSOR, CURSOR_PARAGRAPH, XPATH, REPORT. | None |false| false |None|None|
| `xpath` | XPath expression for reading text at a specific location. | None |Conditional| false |None|Required when `at` = `XPATH`|

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `text` | The retrieved text content. | false |

## Examples

### Get full document text
```yaml
 - step: GET_TEXT_CONTENT
 ```

### Get text at XPath
```yaml
 - step: GET_TEXT_CONTENT
   at: XPATH
   xpath: "/doc/story/content/headline/p"
 ```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
