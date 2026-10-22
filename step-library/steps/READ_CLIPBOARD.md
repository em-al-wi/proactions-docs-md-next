---
sidebar_position: 1
---

# READ_CLIPBOARD

Reads content from the user clipboard. Prefers HTML content when available, falls back to plain text.

## At a glance
- **Category** Browser
- **Version:** 1.0.3
- **Applications:** all
- **Scope:** all

## When to Use

Use **READ_CLIPBOARD** to bring the user clipboard HTML or plain text into a flow.

## Data Flow

Reads no flow context values. Writes clipboard content to `text` or its configured output name.

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `text` | The clipboard content (HTML or plain text) stored in the default text output. | false |

## Examples

### Read clipboard into default text output
```yaml
- step: READ_CLIPBOARD
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
