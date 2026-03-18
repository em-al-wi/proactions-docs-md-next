---
sidebar_position: 1
---

# MONITOR_STATUS

Display status messages in the Flow Monitor during workflow execution.

## At a glance
- **Category** UI
- **Version:** 1.2.0
- **Applications:** all
- **Scope:** all

## When to Use


Use **MONITOR_STATUS** to provide real-time status updates to users during long-running workflows.

**Use MONITOR_STATUS when:**
- You want to inform users about current processing steps
- You're performing operations that take time (API calls, processing)
- You want to provide context about what the AI is doing
- You need to show warnings or important information during execution

**Don't use MONITOR_STATUS when:**
- You want to show final results - use **SHOW_RESPONSE** or **SHOW_NOTIFICATION** instead
- You need user input - use **MONITOR_CONFIRM**, **MONITOR_CHOICE**, or **MONITOR_INPUT**

**Message Types:**
- \`info\` - Informational messages (default)
- \`status\` - Processing status (e.g., "Connecting to API...")
- \`warning\` - Warning messages that need attention
- \`success\` - Success confirmation
- \`error\` - Error messages
  

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `message` | The status message to display. Supports variable resolution with \{\{variable\}\}. | None |false| true |None|None|
| `type` | Type of status message. | info |false| false |None|None|
| `clear` | If true, clears all previous messages before showing this one. | false |false| false |None|None|

## Examples

### Simple status update
```yaml
- step: MONITOR_STATUS
  message: "Connecting to OpenAI API..."
  type: status

- step: OPENAI_COMPLETION
  # ... OpenAI step config

- step: MONITOR_STATUS
  message: "Response received!"
  type: success
  ```

### Show progress through steps
```yaml
- step: MONITOR_STATUS
  message: "Step 1/3: Loading document..."

- step: GET_TEXT_CONTENT

- step: MONITOR_STATUS
  message: "Step 2/3: Analyzing..."

- step: OPENAI_COMPLETION
  prompt: "Analyze: {{flowContext.text}}"

- step: MONITOR_STATUS
  message: "Step 3/3: Complete!"
  type: success
  ```

### Warning message
```yaml
- step: MONITOR_STATUS
  type: warning
  message: "Processing {{flowContext.count}} items. This may take a while."
  ```

### Clear previous messages
```yaml
- step: MONITOR_STATUS
  clear: true
  message: "Starting fresh..."
  ```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
