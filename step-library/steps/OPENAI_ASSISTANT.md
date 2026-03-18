---
sidebar_position: 1
---

# OPENAI_ASSISTANT

Create or reuse an OpenAI assistant. If cfg.assistantId is provided the assistant will be retrieved; otherwise a new assistant is created. The created or retrieved assistant is written to the flow context.

## At a glance
- **Category** LLM
- **Version:** 1.0.9
- **Applications:** all
- **Scope:** all
- **Default Service:** OPENAI_COMPLETION

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `assistantId` | If provided, load the assistant with this ID instead of creating a new one. | None |false| false |None|None|
| `reuse` | If true and storeIn is provided, try to reuse a persisted assistant id. | None |false| false |None|None|
| `storeIn` | Storage location for persisting assistant id: page/session/browser. | None |false| false |None|None|
| `instruction` | Initial instructions / system prompt for the assistant (supports templates). | None |false| true |None|None|
| `model` | Model to use when creating the assistant | None |false| false |None|None|

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `assistant` | Created/retrieved assistant object | false |

## Examples

### Load an existing assistant
```yaml
- step: OPENAI_ASSISTANT
  assistantId: asst_Ja0fm7Rova3QJGbKrAKrt7HS
```

### Create an assistant and persist it
```yaml
- step: OPENAI_ASSISTANT
  instruction: "Be concise and formal"
  model: "gpt-4o"
  reuse: true
  storeIn: "session"
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
