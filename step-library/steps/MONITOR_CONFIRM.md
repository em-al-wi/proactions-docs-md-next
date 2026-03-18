---
sidebar_position: 1
---

# MONITOR_CONFIRM

Display a confirmation dialog in the Flow Monitor. Pauses workflow until user responds.

## At a glance
- **Category** UI
- **Version:** 1.2.0
- **Applications:** all
- **Scope:** all

## When to Use


Use **MONITOR_CONFIRM** when you need Yes/No confirmation before proceeding.

**Use MONITOR_CONFIRM for:**
- Confirming destructive operations (delete, publish, overwrite)
- Approval of AI-generated content
- Simple Yes/No decisions
- Checkpoint confirmations in workflows

**Don't use MONITOR_CONFIRM when:**
- You need multiple choice options → use **MONITOR_CHOICE**
- You only need to show status → use **MONITOR_STATUS**
- You need text input → use **MONITOR_INPUT**
- You just need "Continue" button → use **MONITOR_STATUS** with appropriate message
  

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `message` | The confirmation message to display. Supports variable resolution with \{\{variable\}\}. | None |true| true |None|None|
| `content` | Optional rich content to display (e.g., AI-generated text for review). Supports markdown. | None |false| true |None|None|
| `confirm` | Label for the confirm button (default: "Confirm"). Used in Yes/No mode. | Confirm |false| false |None|None|
| `cancel` | Label for the cancel button (default: "Cancel"). Used in Yes/No mode. | Cancel |false| false |None|None|
| `confirmType` | Visual style of the confirm button. | primary |false| false |None|None|
| `timeout` | Optional timeout in seconds. Auto-cancels if user does not respond. | None |false| false |None|None|

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `boolean` | true if confirmed, false if cancelled. Default output: flowContext.confirmed | false |

## Examples

### Simple confirmation (default output)
```yaml
- step: MONITOR_CONFIRM
  message: "Publish this article?"

- step: IF
  condition: "{{flowContext.confirmed}}"
  then:
    - step: PUBLISH_ARTICLE
  ```

### Confirmation with custom labels
```yaml
- step: MONITOR_CONFIRM
  message: "Publish this article?"
  confirm: "Yes, Publish"
  cancel: "Cancel"
  confirmType: danger

- step: IF
  condition: "{{flowContext.confirmed}}"
  then:
    - step: ... # publish
  ```

### Custom output name
```yaml
- step: MONITOR_CONFIRM
  message: "Delete all drafts?"
  confirm: "Delete"
  cancel: "Keep"
  confirmType: danger
  outputs:
    - type: boolean
      name: shouldDelete

- step: IF
  condition: "{{flowContext.shouldDelete}}"
  then:
    - step: ...
  ```

### AI content approval with preview
```yaml
- step: MONITOR_CONFIRM
  message: "Review the AI response:"
  content: "{{flowContext.aiGeneratedText}}"
  confirm: "Approve"
  cancel: "Reject"
  confirmType: success

- step: IF
  condition: "{{confirmed}}"
  then:
    - step: INSERT_TEXT
    # ...
  else:
    - step: SHOW_NOTIFICATION
      message: "AI response rejected"
  ```

### Confirmation with timeout
```yaml
- step: MONITOR_CONFIRM
  message: "Approve changes?"
  timeout: 30
  outputs:
    - type: boolean
      name: approved

- step: IF
  condition: "{{flowContext.approved}}"
  then:
    - step: ...
  else:
    - step: SHOW_NOTIFICATION
      message: "Timed out or cancelled"
  ```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
