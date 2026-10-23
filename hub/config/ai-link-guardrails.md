---
id: ai-link-guardrails
title: Guardrails
sidebar_label: Guardrails
---

# Guardrails

Guardrails are content checks that run around an AI-Link request: against the outbound messages before the provider is called (`pre`), against the generated text afterwards (`post`), or both. Each violated guardrail can deny the request, redact the offending content, or just record the event.

Guardrails apply to `aiTargets` only — legacy targets use their own `config.beforeRequestHooks` (see [Legacy targets](./ai-link-legacy.md#guardrails)).

## Attaching guardrails to a target

Guardrails are declared once in the top-level `guardrails:` list and attached to a concrete target by name:

```yaml
aiTargets:
  - id: openai-main
    provider: openai
    apiKey: ${OPENAI_API_KEY}
    options:
      model: gpt-6-luna
    guardrails: [no-cc, output-limit]

guardrails:
  - name: no-cc
    type: regex
    phase: pre
    onFail: deny
    patterns: ["\\b\\d{16}\\b"]

  - name: output-limit
    type: length
    phase: post
    onFail: log
    maxChars: 20000
```

- `name` must be unique across the `guardrails` list.
- A target may reference several guardrails; a guardrail may be referenced by several targets.
- Composites cannot declare `guardrails:` directly — attach them to the routes.
- A guardrail's own `target` (for `type: llm`) must be a concrete `aiTargets` entry that does not itself declare guardrails (recursive checks are rejected at startup), and it may not be a composite.

`guardrails` is supported on chat completions and the Responses API. Image generation and audio endpoints have no guardrail coverage.

## Common fields

| Field             | Values                                                | Default                        |
| ----------------- | ----------------------------------------------------- | ------------------------------ |
| `name`            | string, unique                                        | required                       |
| `type`            | `regex`, `length`, `json`, `pii`, `llm`               | required                       |
| `phase`           | `pre`, `post`, `both`                                 | `pre`                          |
| `onFail`          | `deny`, `redact`, `log`                               | `deny`                         |
| `applyTo`         | `last-user-message`, `all-messages`, `output`         | `last-user-message` (pre), `output` (post) |
| `onError`         | `allow`, `deny` (`llm` only)                          | `allow`                        |

Rules enforced at startup:

- `onFail: redact` is valid only for `regex` and `pii` — the other types never produce redacted content.
- `type: json` is post-phase only.
- `applyTo: output` requires `phase: post` or `both`.

## Types

### `regex`

Flags content matching any of the configured patterns. With `onFail: redact`, every match of every matching pattern is replaced with `[REDACTED]`.

```yaml
guardrails:
  - name: no-cc
    type: regex
    phase: pre
    onFail: deny
    patterns: ["\\b\\d{16}\\b"]

  - name: mask-emails
    type: regex
    phase: both
    onFail: redact
    patterns: ["[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}"]
```

An invalid pattern is a check *error*, not a violation. `onError` cannot be set on non-`llm` types, so the request proceeds and the error is counted in `checkErrors`.

### `length`

Flags content exceeding `maxChars` and/or `maxWords`. Never redacts — combine it with `onFail: deny` or `onFail: log`.

```yaml
guardrails:
  - name: output-limit
    type: length
    phase: post
    onFail: deny
    maxChars: 20000
    maxWords: 3000
```

### `json`

Flags output that does not parse as JSON. Post phase only.

```yaml
guardrails:
  - name: must-be-json
    type: json
    phase: post
    onFail: log
```

Pair it with `response_format: json_object` / `json_schema` (see [Structured output](./ai-link.md#structured-output)) when you need the provider to produce JSON in the first place.

### `pii`

Detects built-in PII categories: `email`, `iban`, `credit-card` (Luhn-validated), `phone`. Omitting `categories` runs all of them.

```yaml
guardrails:
  - name: mask-pii
    type: pii
    phase: both
    onFail: redact
    categories: [email, iban]
```

Higher-priority categories claim overlapping spans, so a 16-digit card number is not also reported as a phone number.

### `llm` — judge

Asks a language model whether the content violates a written policy, and gates the verdict on a confidence `score` (a low-confidence "flagged" verdict does not deny).

```yaml
aiTargets:
  - id: brand-safety-target
    provider: openai
    apiKey: ${OPENAI_API_KEY}
    options:
      model: gpt-6-luna

guardrails:
  - name: brand-safety
    type: llm
    mode: judge
    target: brand-safety-target
    model: gpt-6-luna
    phase: both
    onFail: deny
    onError: allow
    threshold: 0.8
    maxContentChars: 8000
    timeout: 10000
    policy: |
      Flag content that mentions competitor products by name.
```

`mode: judge` requires `policy`.

### `llm` — moderation API

Calls an OpenAI-compatible `/moderations` endpoint and flags content whose maximum category score reaches `threshold`, computed over only the categories the API flagged **and** that survive `config.categories` filtering (when set) — not over every category the API returns, and not an average.

```yaml
aiTargets:
  - id: mistral-moderation
    provider: mistral
    apiKey: ${MISTRAL_API_KEY}
    settings:
      baseURL: https://api.mistral.ai/v1

  - id: guarded-chat
    provider: openai
    apiKey: ${OPENAI_API_KEY}
    options:
      model: gpt-6-luna
    guardrails: [moderation]

guardrails:
  - name: moderation
    type: llm
    mode: moderation-api
    target: mistral-moderation
    model: mistral-moderation-latest
    categories: [pii, sexual, hate_and_discrimination]
    threshold: 0.8
    onFail: deny
    onError: allow
```

`mode: moderation-api` forbids `policy` and only works with targets whose provider is `openai`, `openai-compatible`, or `mistral`. An `azure` target passes config validation but rejects at runtime with a check error — its moderation endpoint is not wired up yet.

:::warning[Fail-open by default]
`onError: allow` (the default) means an outage, timeout or auth failure in the judge/moderation call silently disables enforcement for that guardrail. Use `onError: deny` when the check must be fail-closed.
:::

## Actions

| `onFail`  | Effect                                                                                     |
| --------- | ------------------------------------------------------------------------------------------ |
| `deny`    | Aborts the request with HTTP `422` (`guardrail_violation`); nothing is sent upstream/back    |
| `redact`  | Rewrites the flagged content in place and lets the request proceed (`regex`, `pii` only)     |
| `log`     | Records the event and proceeds with the request unchanged                                    |

`onError` (`llm` only) separates a *check execution* failure (the judge model errored, timed out, or returned garbage) from a *violation*: `allow` proceeds, `deny` aborts with `422`.

## Error response

A `deny` action returns HTTP `422` with:

```json
{
  "error": {
    "type": "guardrail_violation",
    "guardrail": "no-cc",
    "phase": "pre",
    "categories": [],
    "reason": "content matched pattern(s): \\b\\d{16}\\b"
  }
}
```

Guardrails run before the response is committed, so a pre-phase deny never opens a stream.

## Streaming behaviour

A post/both guardrail that can still deny or redact (or whose `onError` is `deny`) cannot be enforced against a live stream — bytes already sent cannot be un-sent. For such a target, `stream: true` requests are generated in full internally, checked, and then replayed as a synthesized SSE stream that is wire-indistinguishable from a real one.

Operational impact:

- **No time-to-first-byte benefit.** `stream: true` becomes full-generation latency.
- **The timeout covers the whole generation** — `generateText` plus guardrail time — not just TTFB as for a normal stream. Raise `requestTimeout` on any target that combines streaming with a blocking post guardrail, or a response that would have arrived in time as a real stream can now time out first.

Log-only post guardrails (`onFail: log`, `onError: allow`) do not trigger buffering: they are evaluated after a real stream ends, against the exact text the client already received.

## Limitations

- Post-phase guardrails inspect the generated **text** only — neither tool-call arguments nor reasoning output is covered.
- `applyTo: all-messages` checks each message individually, so a violation split across several messages is not detected. For `llm` guardrails it also means one upstream check call **per message**, so cost and latency scale with conversation length — prefer `last-user-message` on long conversations.
- Image generation, audio speech, audio transcription and audio translation have no guardrail coverage.
- `llm` with `mode: moderation-api` does not support `azure` targets at runtime.

## Monitoring

The [Admin Panel](../admin-panel.md) AI-Link tab lists every configured guardrail with per-guardrail counters: `checks`, `flagged`, `denied`, `redacted`, `logged`, and `checkErrors`. A rising `checkErrors` count with `onError: allow` means enforcement is silently bypassed.
