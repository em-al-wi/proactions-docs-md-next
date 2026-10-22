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

## When to Use

Use **OPENAI_THREAD** to establish a persistent Assistants API conversation thread.

## Data Flow

Reads no flow context values. Writes the created or reused thread to `thread` or its configured output name; it can persist the ID outside flow context.

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
