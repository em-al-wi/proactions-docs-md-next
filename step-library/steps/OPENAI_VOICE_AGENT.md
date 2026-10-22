---
sidebar_position: 1
---

# OPENAI_VOICE_AGENT

Full-duplex voice session using the OpenAI Realtime API over WebRTC (default, with automatic WebSocket fallback). Captures microphone input, streams to the model, plays back the spoken response, and accumulates a conversation transcript. Supports tool calls and structured results.

## At a glance
- **Category** LLM
- **Version:** 1.0.0
- **Applications:** all
- **Scope:** all
- **Default Service:** OPENAI_COMPLETION

## When to Use


Use **OPENAI_VOICE_AGENT** when you need a real-time voice conversation with an OpenAI model.

**Use OPENAI_VOICE_AGENT when:**
- You need a live voice conversation (microphone input + audio playback)
- You want the model to perform actions or call tools during the conversation
- You want a transcript of the conversation for downstream steps

**Don't use OPENAI_VOICE_AGENT when:**
- You only need text input/output — use **OPENAI_COMPLETION** instead
- You need to transcribe audio without a live conversation — use **OPENAI_TRANSCRIPTION**

## Prerequisites

**Required Services:**
- A dedicated OpenAI Realtime service is recommended. It must provide apiKey, a base endpoint without /v1 (for example https://api.openai.com), and a Realtime-capable model.

**Required Context:**

**Platform Requirements:**
- Requires a secure browser context with microphone permission, WebRTC, Web Audio API, AudioWorklet, and MediaDevices.getUserMedia support
- The browser must reach api.openai.com over HTTPS/WSS; WebRTC networks must also permit media negotiation or the step falls back to WebSocket

## Data Flow

Reads resolved behavior and configured tool or skill conditions. Writes the transcript and optional end-session summary to `text`, optionally writes messages and usage outputs, and writes structured end-session results to `object` when supplied.

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `model` | Realtime model to use. Falls back to the service model, then gpt-realtime-2. | None |false| false |None|None|
| `behavior` | System instructions for the voice agent. Supports variable resolution. Voice steps do not inherit service.defaultBehavior, so configure behavior here unless a loaded skill supplies all instructions. | None |false| true |None|None|
| `voice` | Output voice: marin, cedar, alloy, ash, ballad, coral, echo, sage, shimmer, or verse. | marin |false| false |None|None|
| `transport` | Connection transport: 'webrtc' (default — OpenAI's recommendation for browsers: native echo cancellation, automatic barge-in truncation) or 'websocket'. WebRTC falls back to WebSocket automatically when the connection cannot be established (e.g. UDP blocked). | webrtc |false| false |None|None|
| `instruction` | Optional user-prompt passed through to OpenAIRequestBuilder for tool resolution (not sent to the Realtime API) | None |false| false |None|None|
| `vad` | Voice activity detection configuration (type, eagerness, threshold, idleTimeoutMs, bargeIn, etc.). idleTimeoutMs (server_vad only, 5000–30000) makes the model prompt the user after a long silence. For server_vad, raise threshold (default 0.5) to require louder audio in noisy rooms. Set type: 'push_to_talk' to disable server-side VAD entirely — the user explicitly opens/closes the mic via a hold control in the voice bar; the VAD tuning fields are ignored in this mode. bargeIn controls when the user interrupts a speaking agent, in every mode: 'speech' (VAD default) interrupts on detected voice — under push_to_talk once actual voice is heard while holding; 'off' never interrupts (noisy rooms: VAD then answers the captured speech after the response finishes; push_to_talk refuses presses until the agent is done); 'press' (push_to_talk only, its default) cancels immediately on press. | None |false| false |None|None|
| `noiseReduction` | Server-side input noise reduction applied before VAD and the model — reduces background noise falsely triggering the listening state. 'far_field' (default) for laptop/desk/conference mics, 'near_field' for headsets, 'none' to leave unconfigured. | far_field |false| false |None|None|
| `reasoningEffort` | Reasoning effort for reasoning-capable Realtime models (gpt-realtime-2): minimal \| low \| medium \| high \| xhigh \| none. Defaults to low for gpt-realtime-2 models and is omitted for other model families. Use 'none' to omit it explicitly. | None |false| false |None|None|
| `transcriptionModel` | Model for server-side transcription of user speech (default: gpt-4o-transcribe). Options: whisper-1, gpt-4o-transcribe, gpt-4o-mini-transcribe, gpt-4o-transcribe-diarize. | gpt-4o-transcribe |false| false |None|None|
| `maxDurationSecs` | Maximum session duration in seconds before auto-close (default: 300) | 300 |false| false |Range:1-3600|None|
| `showToolCalls` | When true, each tool call is shown in the monitor as a collapsible entry (name + args + result). Useful for debugging and transparency. Default: false. | false |false| false |None|None|
| `toolsReuseContext` | When true and a tool uses steps, the full flowContext is passed to the sub-flow instead of a fresh context with only the tool arguments. | None |false| false |None|None|
| `functionsReuseContext` | Whether to reuse full flowContext for legacy cfg.functions handlers (deprecated, use toolsReuseContext instead). | None |false| false |None|None|
| `tools` | Array of user-defined tool/function descriptors available during the voice session. An optional `when` condition (template expression or boolean, evaluated once at session start) gates whether the tool is exposed to the model. | None |false| false |None|None|
| `functions` | Array of legacy function descriptors for tool calling (deprecated, use tools instead). | None |false| false |None|None|
| `builtinTools` | List of built-in tool classes to enable during the voice session. Object entries support an optional `when` condition (template expression or boolean, evaluated once at session start) that gates whether the tools are exposed to the model. | None |false| false |None|None|
| `autoDiscoverTools` | Enable dynamic tool discovery during the voice session | None |false| false |None|None|
| `toolStrict` | Enable strict validation of builtin tool arguments. | false |false| false |None|None|
| `skills` | List of skill names to load for the voice session. Skill instructions are prepended to the behavior sent to the Realtime API, and skill reference documents become readable via the read_skill_reference tool. Each skill is loaded from `/SysConfig/ProActions/Skills/<skillName>/SKILL.md`. Entries can also be objects `{ name, when }` where `when` (template expression or boolean, evaluated once at session start) gates whether the skill is loaded. | None |false| false |None|None|

### `vad` Object Structure

Voice activity detection configuration (type, eagerness, threshold, idleTimeoutMs, bargeIn, etc.). idleTimeoutMs (server_vad only, 5000–30000) makes the model prompt the user after a long silence. For server_vad, raise threshold (default 0.5) to require louder audio in noisy rooms. Set type: 'push_to_talk' to disable server-side VAD entirely — the user explicitly opens/closes the mic via a hold control in the voice bar; the VAD tuning fields are ignored in this mode. bargeIn controls when the user interrupts a speaking agent, in every mode: 'speech' (VAD default) interrupts on detected voice — under push_to_talk once actual voice is heard while holding; 'off' never interrupts (noisy rooms: VAD then answers the captured speech after the response finishes; push_to_talk refuses presses until the agent is done); 'press' (push_to_talk only, its default) cancels immediately on press.

**Fixed Properties:**

| Property | Type | Description | Required | Constraints |
|---|---|---|:---:|---|
| `type` | `string` | Turn detection mode. | false |Values: semantic_vad, server_vad, push_to_talk|
| `mode` | `string` | Push-to-talk control mode. Only hold is currently supported. | false |Values: hold|
| `bargeIn` | `string` | When user input interrupts a response. press is intended for push_to_talk only. | false |Values: press, speech, off|
| `eagerness` | `string` | semantic_vad only: how eagerly the model ends a turn. | false |Values: auto, low, medium, high|
| `threshold` | `number` | server_vad only: activation threshold in the inclusive range 0-1. | false |Range:0-1|
| `prefixPaddingMs` | `number` | server_vad only: non-negative milliseconds of audio retained before detected speech. | false |Range:0-∞|
| `silenceDurationMs` | `number` | server_vad only: non-negative milliseconds of silence used to detect the end of speech. | false |Range:0-∞|
| `idleTimeoutMs` | `number` | server_vad only: silence before the model prompts the user. | false |Range:5000-30000|

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `text` | Markdown-formatted conversation transcript (alternating **You:** / **Agent:** turns) | false |
| `object` | Structured result passed by the model via end_session(\{ result: ... \}) | true |
| `usage` | Optional token usage accumulated across the voice session | true |
| `messages` | Raw transcript array of TranscriptTurn objects (\{ role, content \}) for downstream steps | true |

## Examples

### Simple voice assistant session
```yaml
- step: OPENAI_VOICE_AGENT
  service: OPENAI_REALTIME
  behavior: |
    You are a helpful editorial voice assistant.

    Complete the user's request, then call end_session with a concise summary.
  voice: marin
  maxDurationSecs: 300
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
