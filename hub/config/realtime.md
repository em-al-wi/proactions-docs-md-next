---
id: realtime
title: Realtime Relay (Voice Agents)
sidebar_label: Realtime Relay
---

# Realtime Relay (Voice Agents)

ProActions `HUB_VOICE_AGENT` steps open a live audio session instead of a request/response completion. The browser cannot reach OpenAI directly in a centralized-key deployment, and a proxy cannot relay WebRTC media, so the Hub provides an authenticated WebSocket relay to the OpenAI Realtime API.

## Overview

The Hub owns exactly two upgrade paths:

- `/realtime` — the canonical path, used when the reverse proxy strips every external prefix.
- `/proactions/realtime` — for deployments whose ingress strips only the app prefix and leaves the ProActions prefix in place.

No deeper prefix is accepted. A request to e.g. `/swing/proactions/realtime` must be rewritten by the ingress before it reaches the Hub.

Realtime is **disabled by default**, both globally and on every target. Enabling it requires all three of:

1. `auth.enabled: true` with `auth.type: token` or `auth.type: edapi`
2. an explicit origin policy
3. a safety-identifier secret of at least 32 bytes

See [Voice Agents](../../guides/configuration/voice-agents.md) for the client side of `HUB_VOICE_AGENT`.

## Prerequisites

- **Main authentication enabled** with `auth.type: token` or `auth.type: edapi`. Realtime authentication is stateless — it uses no HTTP sessions, no cookies, and no admin-key fallback. A `Bearer <non-empty token>` is required in both HTTP and WebSocket credentials, and malformed authorization prefixes are rejected outright.
- **An exact origin policy.** `allowedOrigins` lists the exact origins permitted to open a relay session. CORS configuration does not grant Realtime access.
- **A 32-byte secret.** `safetyIdentifierSecret` must be at least 32 UTF-8 bytes. The Hub derives the provider-side safety identifier from the authenticated principal; it never sends a principal, CMS token, article or action identifier to OpenAI.

```yaml
hub:
  realtime:
    enabled: true
    allowedOrigins:
      - https://swing.your-env.em-cms.cloud
    allowMissingOrigin: false
    safetyIdentifierSecret: ${REALTIME_SAFETY_IDENTIFIER_SECRET}
```

`allowMissingOrigin: true` is only for controlled non-browser clients or tests: it permits an **absent** `Origin` header but never accepts an invalid present one.

## Global policy

The checked-in disabled defaults are the operational reference:

```yaml
hub:
  realtime:
    enabled: false
    allowedOrigins: []
    allowMissingOrigin: false
    safetyIdentifierSecret: ${REALTIME_SAFETY_IDENTIFIER_SECRET}
    admissionTimeoutMs: 9000
    providerConnectTimeoutMs: 8000
    maxActiveSessions: 100
    maxActiveSessionsPerPrincipal: 2
    maxSessionDurationMs: 600000
    maxAllowedSessionDurationMs: 3600000
    idleTimeoutMs: 120000
    maxMessageBytes: 1048576
    maxBytesPerSecond: 2097152
    trafficBurstBytes: 4194304
    pauseBufferedBytes: 524288
    resumeBufferedBytes: 131072
    maxBufferedBytes: 1048576
    backpressureTimeoutMs: 10000
    heartbeatIntervalMs: 30000
    heartbeatTimeoutMs: 10000
    shutdownGraceMs: 5000
```

| Setting                         | Default | Purpose                                                                  |
| ------------------------------- | ------: | ------------------------------------------------------------------------ |
| `admissionTimeoutMs`            |    9000 | Total admission budget, including authentication and provider readiness. |
| `providerConnectTimeoutMs`      |    8000 | Provider-side connection and readiness budget inside admission.          |
| `maxActiveSessions`             |     100 | Global pending plus active admission limit.                              |
| `maxActiveSessionsPerPrincipal` |       2 | Per-principal pending plus active relay session limit.                   |
| `maxSessionDurationMs`          |  600000 | Default maximum relay session duration.                                  |
| `maxAllowedSessionDurationMs`   | 3600000 | Global cap for any effective session duration.                           |
| `idleTimeoutMs`                 |  120000 | Maximum application-idle period; control ping/pong does not reset it.    |
| `maxMessageBytes`               | 1048576 | Maximum complete text or binary message size.                            |
| `maxBytesPerSecond`             | 2097152 | Sustained per-direction byte rate.                                       |
| `trafficBurstBytes`             | 4194304 | Per-direction token-bucket burst allowance.                              |
| `pauseBufferedBytes`            |  524288 | Destination queue high-water threshold that pauses the source.           |
| `resumeBufferedBytes`           |  131072 | Destination queue low-water threshold that resumes the source.           |
| `maxBufferedBytes`              | 1048576 | Destination queue hard limit.                                            |
| `backpressureTimeoutMs`         |   10000 | Maximum continuous high-water duration.                                  |
| `heartbeatIntervalMs`           |   30000 | Client- and provider-side ping interval.                                 |
| `heartbeatTimeoutMs`            |   10000 | Liveness proof deadline after a ping.                                    |
| `shutdownGraceMs`               |    5000 | Maximum Hub shutdown drain period.                                       |

All durations, sizes, rates and counts must be positive safe integers, and the configuration validates the timing, capacity, duration, burst, buffering and heartbeat relationships. A relay session pauses at high-water, resumes below low-water, and only times out while high-water pressure persists.

## Realtime target

A target becomes a **Realtime target** only when it is concrete, has canonical `provider: openai`, has a non-empty top-level `apiKey`, and declares `realtime.enabled: true`:

```yaml
aiTargets:
  - id: openai-realtime
    provider: openai
    apiKey: ${OPENAI_API_KEY}
    options:
      model: gpt-realtime-2.1
    realtime:
      enabled: true
      maxSessionDurationMs: 300000
```

Eligibility:

- Legacy `targets:` entries cannot relay Realtime sessions.
- Composite targets cannot relay Realtime sessions.
- Non-OpenAI providers are not supported.

Target `realtime` values may only **tighten** `maxSessionDurationMs`, `idleTimeoutMs`, `maxMessageBytes` and `maxBytesPerSecond`; they cannot broaden a global policy.

The effective model is the optional client `model` query value first, then the target's `options.model` default. Admission fails when neither is available.

Provider-side endpoints default to `https://api.openai.com/v1`, mapped to `wss://.../realtime?model=...`. Custom HTTPS endpoints are supported; plaintext `http`/`ws` is permitted only for loopback development and test endpoints. Endpoint user information, `settings.apiKey`, and configured overrides of the authorization, WebSocket handshake, OpenAI organization/project or safety headers are rejected. Provider compression is disabled.

:::warning[Rotating credentials]
Restart the Hub after changing a Realtime target's credential. Live credential rotation is not supported.
:::

## Client contract

AI-Kit clients need no change:

```text
/realtime?model=<optional visible-ASCII model>
Sec-WebSocket-Protocol: realtime, proactions-auth.Bearer.<token>, proactions-target.<target>, ...
```

The query accepts only one optional `model`. The protocol must offer exactly one `realtime`, one bearer credential and one target; optional `proactions-article.<label>` and `proactions-action.<label>` entries are bounded correlation labels. The Hub selects only `realtime`.

The Hub delays client-side acceptance until **provider readiness** — the first valid provider-side `session.created` event, which is delivered unchanged as the first client-side event. Admission has a 9 s deadline; AI-Kit expects the browser upgrade within 10 s and provider readiness within 15 s. After acceptance, text and binary frames remain transparent and ordered in both directions.

## Reverse proxy

The ingress in front of the Hub must:

- forward the WebSocket `Upgrade` and `Connection` headers
- disable proxy buffering for these paths
- use long-lived upstream read/send timeouts exceeding the configured session duration
- strip external prefixes so the Hub receives `/realtime` or `/proactions/realtime` — for example rewrite `/swing/proactions-dev/proactions/realtime` to `/proactions/realtime`, or strip the ProActions prefix too for `/realtime`
- not terminate or rewrite the Realtime subprotocol header

## Close codes

| Code   | Meaning                                        |
| ------ | ---------------------------------------------- |
| `1009` | Oversized message                              |
| `1008` | Policy, rate, or backpressure violation        |
| `1011` | Transport failure                              |
| `1012` | Hub restart (server-side shutdown)             |

## Readiness and shutdown

Disabled Realtime is readiness healthy and `skipped`. Enabled Realtime is ready only while its listener remains attached to the actual Hub server and the module is not draining. Readiness never probes OpenAI.

On shutdown the Hub enters draining, rejects new admissions with `503`, cancels pending admissions, closes active client sides with `1012 Service Restart`, closes provider sides, waits at most `shutdownGraceMs`, then terminates leftovers and finalizes capacity and usage records exactly once.

## Diagnostics and privacy

The [Admin Panel](../admin-panel.md) Realtime tab exposes aggregate pending admissions, active relay sessions, admitted/rejected/completed/failed totals, directional bytes, duration summaries, and categorized rejection/close totals.

Realtime usage is accounted separately: token totals split by modality (input text/audio/image, cached, output text/audio), the provider-response `status` distribution, input-audio transcription usage (billed separately from response tokens), and per-target/per-model breakdowns of sessions, responses, tokens, bytes and duration. Token and response aggregates cover closed sessions only; open sessions appear in a live list with request ID, target, model, correlation labels, start time, age and directional bytes.

Lifecycle logs contain the request ID, target, model, bounded article/action correlation labels, phase, duration, byte totals, token totals and the terminal category. Diagnostics and logs **never** expose principals, credentials, safety identifiers, the raw `Sec-WebSocket-Protocol` header, transcripts, instructions, tool arguments, audio, raw events, provider-side content, or peer-supplied reasons.

## Troubleshooting

### `404` on the upgrade

The path the Hub received is not `/realtime` or `/proactions/realtime`. Check the ingress rewrite: a deeper prefix such as `/swing/proactions/realtime` reaches the Hub unchanged and is not a route.

### `401` / `403` on the upgrade

Either main authentication is disabled/mistyped (Realtime requires `auth.type: token` or `edapi`), the bearer credential is missing or malformed, or the proxy stripped the `Sec-WebSocket-Protocol` header so the `proactions-auth.Bearer.<token>` credential never arrives.

### Origin rejected

The browser's exact origin is not in `allowedOrigins`. Entries are exact matches, and CORS settings do not apply to the relay.

### Upstream failure during admission

The Hub could not establish the provider side. Check that the target's `apiKey` is valid, that the target is a concrete `provider: openai` entry with `realtime.enabled: true`, and that the Hub can reach `api.openai.com:443`.

## Next Steps

- [Voice Agents](../../guides/configuration/voice-agents.md) — client-side configuration and flow examples
- [AI-Link Configuration](./ai-link.md)
- [Admin Panel](../admin-panel.md)
