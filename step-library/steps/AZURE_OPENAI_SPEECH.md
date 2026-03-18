---
sidebar_position: 1
---

# AZURE_OPENAI_SPEECH

Azure OpenAI speech step alias.

:::info Documentation Inheritance
Documentation partly inherited from `OPENAI_SPEECH`.
:::

## At a glance
- **Category** LLM
- **Aliases** SERVICE_AZURE_OPENAI_SPEECH
- **Version:** 1.0.6
- **Applications:** all
- **Scope:** all
- **Default Service:** AZURE_OPENAI_SPEECH

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `model` | Model id to use for TTS (e.g. tts-1) | None |false| false |None|None|
| `voice` | Voice id to use for synthesis | None |false| false |None|None|
| `response_format` | Response audio format (e.g. mp3, wav, pcm) | None |false| false |None|None|
| `speed` | Playback speed multiplier (e.g. 1.0) | None |false| false |None|None|
| `mime_type` | Optional MIME type for the produced blob (e.g. audio/mpeg) | None |false| false |None|None|

## Inputs
| Type | Description | Default | Required | Resolved |
|---|---|:---:|:---:|:---:|
| `text` | Text to synthesize. If omitted, the default text input is used. | None | false | false |

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `blob` | Generated audio blob | false |

## Examples

### Synthesize simple text to MP3
```yaml
- step: AZURE_OPENAI_SPEECH
  model: "tts-1"
  voice: "alloy"
  response_format: "mp3"
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
