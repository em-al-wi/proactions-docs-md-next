---
sidebar_position: 1
---

# ELEVENLABS_TTS

Generate speech audio from text using ElevenLabs TTS. Text is taken from the default text input or a flow variable via `in`.

## At a glance
- **Category** Service
- **Version:** 1.0.12
- **Applications:** all
- **Scope:** all
- **Default Service:** ELEVENLABS

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `apiKey` | Override API key from service configuration | None |false| false |None|None|
| `endpoint` | Override endpoint URL from service configuration | None |false| false |None|None|
| `voice_id` | Voice identifier to use for synthesis | None |false| false |None|None|
| `model_id` | TTS model id | None |false| false |None|None|
| `voice_settings` | Optional voice settings object | None |false| false |None|None|
| `output_format` | Desired audio output format (e.g. mp3, wav, pcm) | None |false| false |None|None|
| `language_code` | Language code for the synthesis | None |false| false |None|None|
| `pronunciation_dictionary_locators` | Pronunciation dictionary locators (array or string) - resolved | None |false| false |None|None|
| `seed` | Seed for deterministic generation | None |false| false |None|None|
| `previous_text` | Optional previous_text context | None |false| false |None|None|
| `next_text` | Optional next_text context | None |false| false |None|None|
| `apply_text_normalization` | Apply normalization to text | None |false| false |None|None|
| `apply_language_text_normalization` | Apply language specific normalization | None |false| false |None|None|
| `mime_type` | MIME type to use for the produced blob (e.g. audio/mpeg) | None |false| false |None|None|

## Inputs
| Type | Description | Default | Required | Resolved |
|---|---|:---:|:---:|:---:|
| `text` | Text to synthesize. If omitted, the default text input is used. | None | false | true |

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `blob` | Generated audio blob | false |

## Examples

### Generate speech from text
```yaml
- step: ELEVENLABS_TTS
  voice_id: "21m00Tcm4TlvDq8ikWAM"
  output_format: "mp3"
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
