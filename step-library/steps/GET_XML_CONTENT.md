---
sidebar_position: 1
---

# GET_XML_CONTENT

Retrieves XML content from the document or a specific location and sets it as output.

## At a glance
- **Category** Editor
- **Aliases** CTX_GET_XML_CONTENT
- **Version:** 1.0.1
- **Applications:** all
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `at` | Location to read XML from. Options: CURSOR, CURSOR_PARAGRAPH, XPATH, REPORT. | None |false| false |None|None|
| `xpath` | XPath expression for reading XML at a specific location. | None |Conditional| false |None|Required when `at` = `XPATH`|

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `text` | The retrieved XML content as a string. | false |
| `object` | The retrieved XML content as an object (for REPORT). | true |

## Examples

### Get full document XML
```yaml
 - step: GET_XML_CONTENT
 ```

### Get XML at XPath
```yaml
 - step: GET_XML_CONTENT
   at: XPATH
   xpath: "/doc/story/article/headline/p"
 ```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
