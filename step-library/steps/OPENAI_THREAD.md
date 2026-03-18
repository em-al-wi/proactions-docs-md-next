---
sidebar_position: 1
---

# OPENAI_THREAD

Create or reuse an OpenAI thread. If cfg.reuse and cfg.storeIn are provided the thread id may be reused. The created thread is written to the flow context.

## At a glance
- **Category** LLM
- **Version:** 1.0.9
- **Applications:** all
- **Scope:** all
- **Default Service:** OPENAI_COMPLETION

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `reuse` | If true and storeIn provided, try to reuse a persisted thread id | None |false| false |None|None|
| `storeIn` | Storage location for persisting thread id: page/session/browser. | None |false| false |None|None|

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `thread` | Created thread object | false |

## Examples

### Create a new thread and persist id
```yaml
- step: OPENAI_THREAD
  reuse: true
  storeIn: "session"
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
