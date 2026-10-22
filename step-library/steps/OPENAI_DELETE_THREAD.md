---
sidebar_position: 1
---

# OPENAI_DELETE_THREAD

Delete a thread using its id. Provide the thread object or id via inputs.

## At a glance
- **Category** LLM
- **Version:** 1.0.9
- **Applications:** all
- **Scope:** all
- **Default Service:** OPENAI_COMPLETION

## When to Use

Use **OPENAI_DELETE_THREAD** to clean up a completed thread when it is no longer needed.

## Data Flow

Reads a thread object from `thread` or its configured input name. Does not write flow context.

## Inputs
| Type | Description | Default | Required | Resolved |
|---|---|:---:|:---:|:---:|
| `thread` | Thread object or id to delete | None | true | false |

## Examples

### Delete a thread
```yaml
- step: OPENAI_DELETE_THREAD
  thread: "{{threadObject}}"
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
