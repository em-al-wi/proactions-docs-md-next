---
sidebar_position: 1
---

# SLEEP

Pause execution for a given delay (ms). Useful for pacing flows or waiting for external state changes.

## At a glance
- **Category** Control
- **Version:** 1.0.8
- **Applications:** all
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `delay` | Delay in milliseconds to sleep. Defaults to 1000 ms. | None |false| false |None|None|

## Examples

### Sleep for 2 seconds
```yaml
- step: SLEEP
  delay: 2000
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
