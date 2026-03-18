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
