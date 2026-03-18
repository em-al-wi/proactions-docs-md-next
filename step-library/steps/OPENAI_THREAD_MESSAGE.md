---
sidebar_position: 1
---

# OPENAI_THREAD_MESSAGE

Send a message to a thread/assistant. The assistant and thread inputs are required. The message can be provided via cfg.instruction or via default text input. The assistant response text is stored in the default text output.

## At a glance
- **Category** LLM
- **Version:** 1.0.9
- **Applications:** all
- **Scope:** all
- **Default Service:** OPENAI_COMPLETION

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `instruction` | Optional message instruction; if omitted the default text input is used. | None |false| true |None|None|

## Inputs
| Type | Description | Default | Required | Resolved |
|---|---|:---:|:---:|:---:|
| `assistant` | Assistant object or id used to send the message | None | true | false |
| `thread` | Thread object or id to send the message to | None | true | false |
| `text` | Message text to send to the assistant. If omitted, reads from cfg.instruction or default text input. | None | false | false |

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `text` | Assistant response text (default text output) | false |

## Examples

### Send a message to a thread
```yaml
- step: OPENAI_ASSISTANT # load assistant
  assistantId: asst_Ja0fm2Rova2QJGbKrAKrtXXX
- step: OPENAI_THREAD # load an existing thread
  reuse: true
  storeIn: session
- step: OPENAI_THREAD_MESSAGE
  instruction: |
    Create a list of the key takeaways from the documents provided.
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
