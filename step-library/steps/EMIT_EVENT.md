---
sidebar_position: 1
---

# EMIT_EVENT

Emits a custom internal event that can trigger other actions subscribed via action events.

## At a glance
- **Category** Control
- **Aliases** DISPATCH_EVENT
- **Version:** 1.2.0
- **Applications:** all
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `name` | Event name to emit. Supports variable resolution. | None |true| true |None|None|
| `payload` | Optional event payload object. Supports variable resolution. | None |false| true |None|None|

## Examples

### Emit custom event with payload
```yaml
- step: EMIT_EVENT
  name: "custom.contentProcessed"
  payload:
    summary: "{{ flowContext.text }}"
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
