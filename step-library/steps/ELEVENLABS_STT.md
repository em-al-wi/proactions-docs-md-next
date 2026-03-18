---
sidebar_position: 1
---

# ELEVENLABS_STT

Transcribe audio using ElevenLabs STT. Provides the transcribed text and optional JSON result.

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
| `model_id` | Model id to use for transcription - default: scribe_v1 | None |false| false |None|None|
| `language_code` | Language code for transcription | None |false| false |None|None|
| `tag_audio_events` | Tag audio events option | None |false| false |None|None|
| `num_speakers` | Number of speakers to detect | None |false| false |None|None|
| `timestamps_granularity` | Timestamps granularity | None |false| false |None|None|
| `diarize` | Enable diarization | None |false| false |None|None|
| `additional_formats` | Additional output formats | None |false| false |None|None|
| `file_format` | Input file format (e.g. wav, mp3) | None |false| false |None|None|
| `cloud_storage_url` | URL to fetch input from cloud | None |false| false |None|None|

## Inputs
| Type | Description | Default | Required | Resolved |
|---|---|:---:|:---:|:---:|
| `file` | Audio file or blob to transcribe | None | true | false |

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `text` | Transcribed text | false |
| `json` | Full transcription result (optional) | true |

## Examples

### Transcribe an audio file
```yaml
- step: ELEVENLABS_STT
  model_id: "scribe_v1-1"
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
