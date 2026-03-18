---
sidebar_position: 1
---

# MONITOR_INPUT

Display an inline text input field in the Flow Monitor.

## At a glance
- **Category** UI
- **Version:** 1.2.0
- **Applications:** all
- **Scope:** all

## When to Use


Use **MONITOR_INPUT** for quick inline text input without opening a modal dialog.

**Use MONITOR_INPUT for:**
- Filename entry
- Quick notes or comments
- Simple text corrections
- Short variable values

**Don't use MONITOR_INPUT when:**
- You need multi-line text → use **PROMPT** or **FORM** (modal)
- You need form validation → use **FORM** (modal)
- You only need Yes/No → use **MONITOR_CONFIRM**
- You need selection from options → use **MONITOR_CHOICE**
  

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `message` | The prompt message asking for input. | None |true| true |None|None|
| `placeholder` | Placeholder text shown in the input field. | None |false| true |None|None|
| `default` | Default value pre-filled in the input field. | None |false| true |None|None|
| `required` | If true, empty input is not allowed. | false |false| false |None|None|
| `maxLength` | Maximum number of characters allowed. | None |false| false |None|None|
| `errorMessage` | Error message shown when the field is submitted empty and required is true. | This field is required |false| true |None|None|

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `text` | The text entered by the user. | false |

## Examples

### Filename input
```yaml
- step: MONITOR_INPUT
  message: "Enter filename:"
  placeholder: "document"
  outputs:
    - type: text
      name: filename
  ```

### Required input with default
```yaml
- step: MONITOR_INPUT
  message: "Enter author name:"
  default: "{{flowContext.currentUser}}"
  required: true
  outputs:
    - type: text
      name: author
  ```

### Short comment
```yaml
- step: MONITOR_INPUT
  message: "Add a note (optional):"
  maxLength: 100
  outputs:
    - type: text
      name: note
  ```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
