---
sidebar_position: 1
---

# DEEPL_WRITE

Rephrase/write text using DeepL Write endpoint. Returns the rephrased text in the default text output.

## At a glance
- **Category** Service
- **Aliases** SERVICE_DEEPL_WRITE
- **Version:** 1.0.9
- **Applications:** all
- **Scope:** all
- **Default Service:** DEEPL_WRITE

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `instruction` | Text to be rephrased or written. If omitted, default text input is used. | None |false| true |None|None|
| `target_lang` | Optional target language for writing/rephrasing. | None |false| false |None|None|
| `writing_style` | Optional writing style parameter. | None |false| false |None|None|
| `tone` | Optional tone parameter for the writing API. | None |false| false |None|None|

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `text` | Rewritten text | false |

## Examples

### Rewrite text using DeepL Write
```yaml
- step: DEEPL_WRITE
  instruction: "This is bad grammar"
  tone: "friendly"
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
