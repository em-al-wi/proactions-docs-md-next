---
sidebar_position: 1
---

# SANITIZE

Sanitizes text input. Can strip markdown code blocks, normalize Unicode characters (quotes, dashes, spaces), and validate/repair XML content before forwarding the result to the flow context.

## At a glance
- **Category** Utils
- **Version:** 1.0.12
- **Applications:** all
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `stripMarkdown` | If true, removes markdown code blocks (``` ... ```) from the input before further processing. | false |false| false |None|None|
| `validateAndRepairXml` | If true, attempts to validate and repair XML fragments found in the input. | false |false| false |None|None|
| `normalizeCharacters` | If true, normalizes Unicode characters to their standard equivalents (e.g., curly quotes to straight quotes, various dashes to hyphens, non-breaking spaces to regular spaces). This ensures consistency between AI responses and input text. | false |false| false |None|None|

## Inputs
| Type | Description | Default | Required | Resolved |
|---|---|:---:|:---:|:---:|
| `text` | Text input to sanitize | None | true | false |

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `text` | Sanitized text output after processing. | false |

## Examples

### Sanitize by stripping markdown code blocks
```yaml
- step: SANITIZE
  stripMarkdown: true
```

### Sanitize and validate/repair XML
```yaml
- step: SANITIZE
  validateAndRepairXml: true
```

### Normalize Unicode characters
```yaml
- step: SANITIZE
  normalizeCharacters: true
```

### Combined sanitization
```yaml
- step: SANITIZE
  stripMarkdown: true
  normalizeCharacters: true
  validateAndRepairXml: true
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
