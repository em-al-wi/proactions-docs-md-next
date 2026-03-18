---
sidebar_position: 1
---

# OPENAI_TRANSCRIPTION

Transcribe audio using OpenAI (or Azure/Hub variants). Reads an audio file/blob from inputs and writes the transcription text and optional segment list.

## At a glance
- **Category** LLM
- **Aliases** SERVICE_OPENAI_TRANSCRIPTION
- **Version:** 1.0.0
- **Applications:** all
- **Scope:** all
- **Default Service:** OPENAI_TRANSCRIPTION

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `instruction` | Optional prompt/instruction to bias transcription. | None |false| true |None|None|
| `language` | Optional language code to hint at language for transcription. | None |false| false |None|None|
| `temperature` | Optional temperature / randomness parameter for transcription model. | None |false| false |None|None|
| `model` | Optional model override (e.g. 'whisper-1'). | None |false| false |None|None|

## Inputs
| Type | Description | Default | Required | Resolved |
|---|---|:---:|:---:|:---:|
| `file` | Audio file or Blob to transcribe | None | true | false |

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `text` | The full transcribed text | false |
| `list` | Optional list of segment texts | true |

## Examples

### Transcribe an uploaded audio file
```yaml
- step: OPENAI_TRANSCRIPTION
  model: "whisper-1"
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
