# Voice Agents

This guide covers `OPENAI_VOICE_AGENT` and `HUB_VOICE_AGENT` — full-duplex voice sessions against the OpenAI Realtime API. The model listens to the microphone, speaks its responses out loud, can call tools and skills mid-conversation, and hands back a transcript (and optionally structured data) when the session ends.

## Overview

A voice agent step opens a live audio session instead of a single request/response completion:

```yaml
- step: OPENAI_VOICE_AGENT
  service: OPENAI_REALTIME
  behavior: |
    You are a helpful editorial voice assistant.

    After completing a task, ask "Is there anything else I can help you with?"
    Keep the conversation going until the user explicitly says they are done.

    Only call end_session after: (1) you have spoken all results verbally,
    (2) the user confirms they are done. Never call it after just one task.
  voice: marin
  maxDurationSecs: 300
```

The step runs until the model calls `end_session`, the user clicks **End session** in the pinned voice bar, or `maxDurationSecs` elapses (default 300s). When it finishes, the transcript is written to `flowContext.text` and any structured result the model provided to `flowContext.object`, so downstream steps can act on the conversation like any other completion.

**`OPENAI_VOICE_AGENT`** connects directly using a short-lived ephemeral key from your OpenAI service configuration. **`HUB_VOICE_AGENT`** is identical but routes through ProActions Hub over WebSocket (a proxy cannot relay WebRTC media), for centralized key management without exposing OpenAI credentials to the browser.

**When to use a voice agent:**
- You need a live, spoken conversation — microphone in, audio out
- The model should perform actions or call tools while talking
- You want a transcript for downstream steps (summaries, logging, structured extraction)

**When not to:** text-only interaction (use `OPENAI_COMPLETION`/`HUB_COMPLETION`), or transcribing audio without a live back-and-forth (use `OPENAI_TRANSCRIPTION`).

## Before You Start

Voice agents require all of the following:

- A secure browser context (`https://`) with microphone permission
- `MediaDevices.getUserMedia`, Web Audio API, and `AudioWorklet` support
- The Flow Monitor enabled, because the pinned Voice Bar contains the session controls
- A Realtime-capable OpenAI model, such as `gpt-realtime-2.1`
- Network and Content Security Policy (CSP) rules that permit the connections described below

For direct OpenAI sessions, the browser needs HTTPS and WSS access to `api.openai.com`; WebRTC media must also be permitted for the preferred transport. If WebRTC negotiation fails, the step falls back to WebSocket.

For Hub sessions, Swing or Prime must provide an EDAPI token. The browser needs WSS access to the Hub relay, and Hub needs WSS egress to `api.openai.com:443`. Standalone clients cannot run `HUB_VOICE_AGENT` because they do not provide the required EDAPI token.

## Configure a Service

Use a dedicated service entry for voice agents. A voice endpoint is a **base URL**: the step appends `/v1/realtime/...` for direct OpenAI or `/proactions/realtime` for Hub. Do not reuse a completion service whose endpoint already ends in `/v1` or `/proactions`.

### Hub (Recommended)

Hub keeps the OpenAI API key on the server. Define a dedicated service in `/SysConfig/ProActions/services.ai-kit.services.yaml`:

```yaml
HUB_REALTIME:
  # The step appends /proactions/realtime, producing
  # {BASE_URL}/swing/proactions/realtime in a standard Swing deployment.
  endpoint: '{BASE_URL}/swing'
  target: openai-realtime
  model: gpt-realtime-2.1
```

Then reference it from the step:

```yaml
- step: HUB_VOICE_AGENT
  service: HUB_REALTIME
  behavior: |
    You are a helpful voice assistant.
    Ask whether the user needs anything else before calling end_session.
```

The `target` must identify an OpenAI Realtime-capable target in Hub. Claude, Gemini, and ordinary chat-completion targets are not interchangeable with a Realtime target.

The Hub administrator must configure the matching target and deploy a Hub version that provides the Realtime WebSocket relay — requires ProActions Hub 1.3.0 or later with `hub.realtime` enabled. See [Realtime Relay](../../hub/config/realtime.md).

```yaml
# ProActions Hub config.yaml
aiTargets:
  - id: openai-realtime
    provider: openai
    apiKey: ${OPENAI_API_KEY}
    options:
      model: gpt-realtime-2.1
    realtime:
      enabled: true
```

The deployment must expose `WSS /proactions/realtime`, authenticate the EDAPI or Hub token sent in `Sec-WebSocket-Protocol`, preserve the requested WebSocket subprotocols, and forward the selected target to OpenAI Realtime. If a reverse proxy sits in front of Hub, configure WebSocket upgrades, trusted TLS, and a connection timeout long enough for `maxDurationSecs`. Confirm relay availability with your Hub administrator; a Hub installation that only exposes the HTTP endpoints under `/v1` does not support `HUB_VOICE_AGENT`.

At session start, AI-Kit automatically verifies that the WebSocket upgrade selects the `realtime` subprotocol and that Hub creates the upstream OpenAI session within 15 seconds. It fails before opening the microphone when the relay route, authentication, proxy subprotocol forwarding, target credentials, or Hub egress are not ready.

### Direct OpenAI

Direct mode loads the long-lived OpenAI API key into browser-delivered ProActions configuration so it can request a short-lived session key. The audio session uses the short-lived key, but this does **not** prevent a browser user from inspecting the configured long-lived key. Prefer Hub for production use.

```yaml
# /SysConfig/ProActions/services.ai-kit.services.yaml
OPENAI_REALTIME:
  apiKey: 'sk-...'
  # Base URL only: do not append /v1.
  endpoint: 'https://api.openai.com'
  model: gpt-realtime-2.1
```

```yaml
- step: OPENAI_VOICE_AGENT
  service: OPENAI_REALTIME
  behavior: |
    You are a helpful voice assistant.
    Ask whether the user needs anything else before calling end_session.
```

`OPENAI_VOICE_AGENT` defaults to a service named `OPENAI_COMPLETION` for compatibility. Only rely on that default when the service already uses the base endpoint `https://api.openai.com` and a Realtime model. A typical completion endpoint such as `https://api.openai.com/v1` is not valid for this step.

Voice steps resolve `behavior` from the step itself and do not inherit `defaultBehavior` from the service. Configure `behavior` explicitly unless a loaded skill supplies all required instructions, including when to call `end_session`.

## Transport: WebRTC vs WebSocket

`OPENAI_VOICE_AGENT` defaults to **WebRTC** — OpenAI's recommendation for browser clients, with native echo cancellation and automatic barge-in truncation. If the WebRTC handshake can't complete (e.g. UDP blocked by a firewall), the step automatically falls back to WebSocket; a warning is posted to the monitor when this happens.

```yaml
- step: OPENAI_VOICE_AGENT
  transport: websocket   # force WebSocket instead of the WebRTC/auto-fallback default
```

`HUB_VOICE_AGENT` always uses WebSocket — the Hub proxy relays the connection, and WebRTC media can't be proxied.

Microphone-permission and hardware errors (denied access, no device found) are surfaced as clear, localized messages and are **not** retried over the fallback transport — the WebSocket path needs the same microphone, so silently retrying would only replace an explicit permission error with a confusing timeout.

## Voice Activity Detection (VAD)

By default the model decides when the user has finished speaking (`semantic_vad`). Tune it via `vad:`:

```yaml
- step: OPENAI_VOICE_AGENT
  vad:
    type: semantic_vad     # or server_vad
    eagerness: low          # semantic_vad only — how quickly it ends a turn
    bargeIn: speech         # default — the user's voice interrupts the agent;
                            # set 'off' for noisy rooms (speech is still
                            # captured and answered after the response ends)
```

For noisy environments, `server_vad` gives you explicit silence-duration control:

```yaml
  vad:
    type: server_vad
    threshold: 0.6          # raise for noisy rooms (default 0.5)
    prefixPaddingMs: 300
    silenceDurationMs: 500
    idleTimeoutMs: 15000     # prompt the user after 15s of silence (server_vad only, 5000-30000)
```

Background noise is additionally filtered server-side before it reaches VAD or the model:

```yaml
  noiseReduction: far_field   # default — laptop/desk/conference mics
  # noiseReduction: near_field  # headset users
  # noiseReduction: none        # leave unconfigured
```

### Push-to-Talk

Set `type: push_to_talk` to disable server-side VAD entirely. The model never decides
when the user is done speaking — the user explicitly opens and closes the mic via a
hold control in the voice bar:

```yaml
  vad:
    type: push_to_talk
```

The VAD tuning fields (`eagerness`, `threshold`, `prefixPaddingMs`, `silenceDurationMs`,
`idleTimeoutMs`) are ignored in this mode — there is no automatic turn detection to
tune. `bargeIn` applies in every mode (see below).

**Controls** — the voice bar renders a single hold button instead of the mute toggle:

- **Mouse**: press and hold to talk, release to send
- **Touch**: tap and hold, lift to send
- **Keyboard**: hold <kbd>Space</kbd> while the monitor panel has focus, release to
  send — clicking anywhere in the monitor (the transcript, the bar) focuses it.
  <kbd>Space</kbd> keeps its normal behavior in the monitor's text inputs and on
  focused buttons, and is never captured while you are typing in the editor.

**Barge-in while the agent is speaking** is controlled by `bargeIn`:

```yaml
  vad:
    type: push_to_talk
    bargeIn: press   # press (default) | speech | off
```

- **`press`** (default) — pressing the hold control interrupts playback immediately
  and opens the mic. The press gesture doubles as barge-in; there is no separate
  interrupt affordance.
- **`speech`** — the agent keeps talking while you hold, and is cut off the moment
  your voice is actually detected on the microphone. Resting a finger on the control
  (or an accidental grab) never interrupts. When microphone analysis is available,
  a hold with no detected speech is discarded instead of being transcribed.
- **`off`** — never interrupt. Presses while the agent is responding are refused:
  the mic stays closed and the release does nothing. Wait for the
  *Ready — hold the button to talk* label, then press. Use this when the agent's
  answer must always be heard in full (e.g. spoken confirmations).

`bargeIn` is the same field in every mode: under `semantic_vad`/`server_vad` the
default is `speech` (the server's own barge-in trigger) and `off` lets the agent
finish while your speech is captured and answered afterwards; `press` exists only
under `push_to_talk`, where it is the default.

**Short presses**: a hold shorter than ~300ms is treated as an accidental tap — the
buffered audio is discarded instead of sent, so the agent won't respond to a quick
brush of the button.

**Between presses** the bar's label tracks the session: *Processing…* / *Responding…*
while the agent works on your turn, and *Ready — hold the button to talk* once it is
your turn again. Wait for the ready label before pressing again — pressing while the
agent is responding is the barge-in gesture and cancels the rest of its answer.

A complete push-to-talk step — everything outside `vad:` works exactly as in the
other examples in this guide:

```yaml
- step: OPENAI_VOICE_AGENT          # or HUB_VOICE_AGENT
  behavior: |
    You are a helpful editorial voice assistant.
    Only call end_session after you have spoken all results and the user
    confirms they are done.
  vad:
    type: push_to_talk
    bargeIn: speech                 # agent yields as soon as the user's voice is heard;
                                    # omit for the default (press interrupts immediately)
  maxDurationSecs: 300
  builtinTools:
    - class: StoryEditor
      config:
        fields:
          headline: '/doc/story/grouphead/headline/p'
```

Use this mode for noisy or shared environments where automatic VAD would trigger
falsely, or where explicit control over when the model listens is required.

## Model, Voice, and Reasoning

```yaml
- step: OPENAI_VOICE_AGENT
  model: gpt-realtime-2.1             # falls back to the service config's model, then gpt-realtime-2
  voice: marin                        # output voice (default: marin)
  reasoningEffort: low                # reasoning-capable models only (gpt-realtime-2 family); default: low
  transcriptionModel: gpt-4o-transcribe  # server-side transcription of the user's speech
```

`reasoningEffort: low` is OpenAI's recommendation for production voice agents — it keeps latency down while retaining basic reasoning. Use `none` to omit the field entirely for non-reasoning models.

## Tools, Skills, and Configuration

Voice agents resolve `tools`, `builtinTools`, `skills`, `when:` conditions, and `config:` blocks through the **same request builder** as `OPENAI_COMPLETION` — anything you already know about tool calling applies here, with one difference: everything is resolved **once, when the session starts**, not per turn.

```yaml
- step: OPENAI_VOICE_AGENT
  behavior: |
    You are a helpful editorial voice assistant.
    ...
  skills:
    - editorial-style
  builtinTools:
    - class: StoryEditor
      config:
        fields:
          headline: '/doc/story/grouphead/headline/p'
          summary: '/doc/story/summary/p'
    - alias: web_search
      target: mcp.brave-search-server.brave_web_search
      args:
        ui_lang: en-US
  autoDiscoverTools: true
```

- **Skills** loaded via `skills:` are prepended to the session's `behavior` before it is sent to the Realtime API, and `read_skill_reference`/`read_skill_instruction` become callable tools exactly as in a completion step. See the [Agent Skills guide](./ai-integration.md#agent-skills).
- **`when:` conditions** and **`config:` blocks** on `builtinTools` entries are evaluated once at session start against the editor/flow state at that moment — see [Conditional Tool Exposure](./ai-integration.md#conditional-tool-exposure-when) and [Configuring Built-in Tools](./ai-integration.md#configuring-built-in-tools-config). A read-only document, for example, can drop all writing tools from the session entirely.
- **`autoDiscoverTools: true`** works during a live session: when the model calls `activateTools`, the newly resolved tools are added to the session's tool list and pushed to the running connection via a Realtime `session.update` — the model can use them in its very next turn without restarting the session.
- **`showToolCalls: true`** displays each tool call in the monitor as a collapsible entry (name, arguments, result) — useful while developing a voice agent's tool set.

```yaml
  showToolCalls: true
  toolsReuseContext: true   # pass the full flowContext to tool sub-flows, not just the tool arguments
```

User-defined `tools:` (script/template/steps-based function calling) work exactly as documented in [Tool Calling](./ai-integration.md#tool-calling-function-calling) — the same `script`, `scriptRef`, `template`, and `steps` handler forms are supported.

## Ending the Session

The model signals completion by calling the built-in `end_session` tool — always present, and reserved: a user-defined tool named `end_session` in your config is replaced by the built-in one, with a warning, since the bridge intercepts that name before user-tool dispatch.

Your `behavior` decides when the conversation is complete. The model can pass a concise human-readable `summary` and an optional machine-readable `result` when it calls `end_session`:

```yaml
  behavior: |
    ...
    Complete the requested editorial task, then call end_session with a concise summary.
```

The summary is appended to the Step's `text` output. If the model passes a `result` object, it is written to `flowContext.object` by default. An `outputs` mapping changes the destination key:

```yaml
- step: OPENAI_VOICE_AGENT
  behavior: |
    Extract the key facts the user dictates. When done, call end_session
    with a short summary and result containing the structured facts.
  outputs:
    - type: object
      name: extractedFacts
```

The user can end the session immediately with **End session** in the pinned voice bar. At `maxDurationSecs` (default 300), the Step asks the model to finish and call `end_session`; if it has not completed after a ten-second grace period, the Step force-closes while preserving the transcript and usage collected so far.

## Outputs

| Output | Description |
|---|---|
| `text` | Markdown-formatted transcript, alternating `**You:**` / `**Agent:**` turns |
| `object` | Structured result passed via `end_session({ result: ... })` — only written if the model provided one |
| `messages` | Raw transcript array of `{ role, content }` turns, for downstream steps (only written when included in `outputs:`) |
| `usage` | Token usage accumulated across completed responses (only written when included in `outputs:`) |

`text` is always written and defaults to `flowContext.text`. `object` is written only when `end_session` supplies a result. `messages` and `usage` are opt-in and are written only when included in `outputs`. Every configured `name` replaces the default flow-context key.

```yaml
- step: OPENAI_VOICE_AGENT
  behavior: '...'
  outputs:
    - type: text
      name: text
    - type: object
      name: object
    - type: messages
      name: conversationTurns
    - type: usage
      name: voiceUsage

- step: MONITOR_STATUS
  type: success
  message: |
    # Session Summary
    {{ flowContext.text }}
```

## Complete Hub Example

Add the service to `/SysConfig/ProActions/services.ai-kit.services.yaml`:

```yaml
HUB_REALTIME:
  endpoint: '{BASE_URL}/swing'
  target: openai-realtime
  model: gpt-realtime-2.1
```

Create `/SysConfig/ProActions/sections/voice-assistant.ai-kit.section.yaml`:

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/em-al-wi/proactions-schema/main/schema/partial-section.schema.json
section: Voice Tools
context: story
monitor:
  # Voice controls are pinned inside the Flow Monitor.
  enabled: true
actions:
  - title: Voice Assistant
    flow:
      - step: HUB_VOICE_AGENT
        monitorLabel: Voice Assistant
        service: HUB_REALTIME
        model: gpt-realtime-2.1
        voice: marin
        behavior: |
          You are a helpful editorial voice assistant.

          Read the story before answering questions about it or editing it.
          After completing a task, ask whether the user needs anything else.
          Keep the conversation open until the user explicitly says they are done.
          Speak all results before calling end_session with a concise summary.
        builtinTools:
          - class: StoryEditor
            config:
              fields:
                headline: '/doc/story/grouphead/headline/p'
                summary: '/doc/story/summary/p'
        maxDurationSecs: 1200
        showToolCalls: true
        outputs:
          - type: messages
            name: voiceConversation
          - type: usage
            name: voiceUsage

    onError:
      - step: SHOW_NOTIFICATION
        notificationType: error
        message: 'Voice assistant failed: {{ flowContext.error }}'
```

Include the section from `/SysConfig/pro-actions.ai-kit.yaml`:

```yaml
AI_KIT:
  SERVICES:
    !include /SysConfig/ProActions/services.ai-kit.services.yaml
  SECTIONS:
    - !include /SysConfig/ProActions/sections/voice-assistant.ai-kit.section.yaml
```

To use direct OpenAI instead, define `OPENAI_REALTIME` as shown above and replace the step with:

```yaml
- step: OPENAI_VOICE_AGENT
  service: OPENAI_REALTIME
  # WebRTC is the default; use transport: websocket only when explicitly required.
```

## Deployment Checklist

- The Voice Agents guide appears under **Guides > Configuration** and the action appears in the expected CMS context.
- The browser is in a secure context and microphone permission is granted.
- The Flow Monitor is enabled and shows the pinned Voice Bar.
- The configured endpoint is a base URL and does not already end in `/v1` or `/proactions`.
- The configured model and Hub target support OpenAI Realtime.
- Direct mode can reach OpenAI over HTTPS/WSS and WebRTC; Hub mode can reach its WebSocket relay.
- Hub mode runs in Swing or Prime with an available EDAPI token.
- Reverse proxies preserve WebSocket upgrades and subprotocols, and CSP `connect-src` permits the required WSS origin.
- The behavior tells the model when to call `end_session`.
- `StoryEditor` remains available on read-only documents so the agent can read them; its write tools return `document_read_only` without changing content.

## Troubleshooting

| Symptom | Check |
|---|---|
| Request URL contains `/v1/v1/realtime` | Use an OpenAI base endpoint without `/v1`, such as `https://api.openai.com`. |
| Hub URL contains `/proactions/proactions/realtime` | Use the Hub base endpoint before `/proactions`; in a standard Swing deployment this is `{BASE_URL}/swing`. |
| Hub returns 404 or the WebSocket never upgrades | The deployed Hub or reverse proxy does not expose `/proactions/realtime`; contact the Hub administrator. |
| Hub authentication fails | Run in Swing/Prime and confirm an EDAPI token is available and WebSocket subprotocol headers are preserved. |
| Direct session uses the wrong model | Set `model: gpt-realtime-2.1` in the dedicated service or step. |
| Microphone access fails | Use HTTPS, grant site permission, verify an input device exists, and check browser policy. |
| WebRTC falls back to WebSocket | Check UDP/media negotiation, firewall policy, and browser diagnostics. The fallback is expected when WebRTC cannot connect. |
| Voice Bar does not appear | Enable the Flow Monitor for the action/section. |
| Agent never ends the session | Add explicit `end_session` conditions to `behavior`, or use **End session** in the Voice Bar. |

To verify a Hub deployment, run the example flow with browser developer tools open. The Network panel must show a WebSocket request ending in `/proactions/realtime?model=...` with status `101`, and the selected subprotocol must be `realtime`. Hub logs must show the authenticated CMS user, selected Realtime target, and an upstream connection to OpenAI. A `404` means the relay is unavailable; `401`/`403` indicates EDAPI authentication or subprotocol forwarding; an upstream connection error indicates target credentials or Hub egress. Use the Hub release supplied by your administrator that explicitly includes this relay; the AI-Kit repository does not define a standalone Hub version number.

See [Service Configuration](./service-configuration.md), [AI Integration](./ai-integration.md), [`OPENAI_VOICE_AGENT`](../../step-library/steps/OPENAI_VOICE_AGENT.md), and [`HUB_VOICE_AGENT`](../../step-library/steps/HUB_VOICE_AGENT.md) for related configuration details.
