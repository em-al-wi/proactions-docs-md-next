---
sidebar_position: 1
---

# OPENAI_SPEECH

Generate speech audio using the OpenAI Speech API (or corresponding Hub/Azure variants). Reads text from the default text input and returns an audio blob.

## At a glance
- **Category** LLM
- **Aliases** SERVICE_OPENAI_SPEECH
- **Version:** 1.0.6
- **Applications:** all
- **Scope:** all
- **Default Service:** OPENAI_SPEECH

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
- step: OPENAI_SPEECH
  model: "tts-1"
  voice: "alloy"
  response_format: "mp3"
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
