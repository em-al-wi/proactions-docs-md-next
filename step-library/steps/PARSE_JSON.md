---
sidebar_position: 1
---

# PARSE_JSON

Parses JSON content from the text input and stores the resulting object in the flow context.

## At a glance
- **Category** Utils
- **Aliases** PROCESS_PARSE_JSON
- **Version:** 1.0.0
- **Applications:** all
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `in` | Optional path or variable name to read the JSON string from the flowContext. If omitted, the default text input is used. | None |false| true |None|None|

## Inputs
| Type | Description | Default | Required | Resolved |
|---|---|:---:|:---:|:---:|
| `text` | JSON string to parse. If omitted, reads from the `in` parameter or default text input. | None | true | false |

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `object` | The parsed JSON object stored in the flow context. | false |
| `text` | The original JSON text string (passthrough). | false |

## Examples

### Parse JSON from text input
```yaml
- step: PARSE_JSON
  in: "rawJson"
```

### Parse JSON from REST response
```yaml
- step: REST    
  method: GET    
  url: 'https://my-api.com/v1/?...'
- step: PARSE_JSON    
  in : "{{ flowContext.text | safe }}"
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
