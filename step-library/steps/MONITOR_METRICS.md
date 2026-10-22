---
sidebar_position: 1
---

# MONITOR_METRICS

Display metrics in the Flow Monitor.

## At a glance
- **Category** UI
- **Version:** 1.0.0
- **Applications:** all
- **Scope:** all

## When to Use

Use **MONITOR_METRICS** to display numeric or labeled evaluation metrics such as scores, counts, confidence, or token usage.

## Data Flow

Reads flow context values referenced by resolved `data` and `title`. Does not write flow context.

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `data` | Metrics as object or array of label/value pairs. | None |true| true |None|None|
| `title` | Optional title displayed above metrics. | None |false| true |None|None|

## Examples

### Render metrics
```yaml
- step: MONITOR_METRICS
  title: "Evaluation"
  data:
    score: 87
    confidence: 0.92
    tokens: 1420
  ```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
