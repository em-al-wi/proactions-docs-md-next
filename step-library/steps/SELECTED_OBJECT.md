---
sidebar_position: 1
---

# SELECTED_OBJECT

Retrieves information about the currently selected content object and sets it as output.

## At a glance
- **Category** Editor
- **Aliases** CTX_SELECTED_OBJECT
- **Version:** 1.0.4
- **Applications:** all
- **Scope:** all

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `text` | The ID of the selected content. | false |
| `object` | The full information object of the selected content. | true |

## Examples

### Get selected content ID
```yaml
 - step: SELECTED_OBJECT
 ```

### Get selected content info
```yaml
 - step: SELECTED_OBJECT
   outputs:
     - type: text
       name: selectedId
     - type: object
       name: selectedInfo
 ```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
