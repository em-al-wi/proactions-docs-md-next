---
sidebar_position: 1
---

# TO_LIST

Convert a textual AI response into a list of strings. Detects JSON arrays, ordered/unordered lists, comma-separated values, or falls back to a single-item array.

## At a glance
- **Category** Utils
- **Aliases** PROCESS_TO_LIST
- **Version:** 1.0.0
- **Applications:** all
- **Scope:** all

## Inputs
| Type | Description | Default | Required | Resolved |
|---|---|:---:|:---:|:---:|
| `text` | Text input to convert to a list (supports JSON arrays, markdown lists, CSV, etc.) | None | true | false |

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `list` | Array of extracted list items (strings) | false |

## Examples

### Convert chat response to list
```yaml
- step: SET
  text: "1. First item\\n2. Second item\\n3. Third item"

- step: TO_LIST
```

### Convert a JSON response containing a list
```yaml
- step: SET
  text: '{"items": ["A","B","C"]}'

- step: TO_LIST
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
