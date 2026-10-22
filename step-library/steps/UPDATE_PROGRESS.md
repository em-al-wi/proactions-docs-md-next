---
sidebar_position: 1
---

# UPDATE_PROGRESS

Update an existing progress bar. Provide percentage (0-100) and optional status text.

## At a glance
- **Category** UI
- **Version:** 1.0.8
- **Applications:** all
- **Scope:** all

## When to Use

Use **UPDATE_PROGRESS** to update the percentage or status of an already displayed progress bar.

## Data Flow

Reads resolved percentage and status values from flow context or config. Does not write flow context.

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `status` | Status text to update on the progress bar. Supports variable resolution. | None |false| true |None|None|
| `percentage` | Progress percentage (0-100). Can be a number or a string that resolves to a number. | None |true| true |None|None|

## Examples

### Update progress
```yaml
- step: UPDATE_PROGRESS
  percentage: "{{progressPercent}}"
  status: "Step {{progressStep}}/5"
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
