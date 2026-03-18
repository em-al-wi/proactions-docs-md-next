---
sidebar_position: 1
---

# MONITOR_CHOICE

Display a selection of choices in the Flow Monitor. Pauses workflow until user selects.

## At a glance
- **Category** UI
- **Version:** 1.2.0
- **Applications:** all
- **Scope:** all

## When to Use


Use **MONITOR_CHOICE** when you need the user to select from a list of predefined options.

**Use MONITOR_CHOICE for:**
- Format selection (PDF, Word, HTML)
- Language/locale selection
- Mode or quality level selection
- Any multiple-choice decision

**Don't use MONITOR_CHOICE when:**
- You only need Yes/No → use **MONITOR_CONFIRM**
- You need text input → use **MONITOR_INPUT**
- You need a complex form → use **FORM** step (modal)
  

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `message` | The prompt message asking user to make a selection. | None |true| true |None|None|
| `choices` | Array of choices. Each choice has: id (returned value), label (display text), optional description. | None |true| false |None|None|
| `default` | ID (or array of IDs in multi mode) of default/pre-selected choice(s). | None |false| false |None|None|
| `selectionMode` | Selection mode: single (default) or multi. | single |false| false |None|None|
| `minSelect` | Minimum selections required in multi mode. | None |false| false |None|None|
| `maxSelect` | Maximum selections allowed in multi mode. | None |false| false |None|None|
| `customInput` | Optional custom text input below options. | None |false| false |None|None|
| `allowCancel` | If true, shows a Cancel option that returns empty string. | true |false| false |None|None|

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `text` | The selected choice ID in single mode (or custom input when no choice selected). | false |
| `object` | Structured selection result in multi mode: \{ selectedIds, customInput \} | false |
| `list` | Selected choice IDs as list in multi mode. | false |

## Examples

### Format selection (default output)
```yaml
- step: MONITOR_CHOICE
  message: "Select export format:"
  choices:
    - id: pdf
      label: "📄 PDF Document"
    - id: docx
      label: "📝 Word Document"
    - id: html
      label: "🌐 HTML Page"

- step: EXPORT_DOCUMENT
  format: "{{flowContext.text}}"
  ```

### Format selection (custom output name)
```yaml
- step: MONITOR_CHOICE
  message: "Select export format:"
  choices:
    - id: pdf
      label: "📄 PDF Document"
    - id: docx
      label: "📝 Word Document"
  outputs:
    - type: text
      name: format

- step: EXPORT_DOCUMENT
  format: "{{flowContext.format}}"
  ```

### Language selection (no cancel)
```yaml
- step: MONITOR_CHOICE
  message: "Translate to:"
  choices:
    - id: de
      label: "🇩🇪 German"
    - id: fr
      label: "🇫🇷 French"
    - id: es
      label: "🇪🇸 Spanish"
  allowCancel: false

- step: TRANSLATE
  targetLanguage: "{{flowContext.text}}"
  ```

### Quality level with default
```yaml
- step: MONITOR_CHOICE
  message: "Select AI quality:"
  choices:
    - id: fast
      label: "⚡ Fast (GPT-3.5)"
    - id: balanced
      label: "⚖️ Balanced (GPT-4)"
    - id: best
      label: "🎯 Best (GPT-4 Turbo)"
  default: balanced

- step: OPENAI_COMPLETION
  model: "{{flowContext.text}}"
  instruction: "Process this text"
  ```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
