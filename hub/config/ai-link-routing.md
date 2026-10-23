---
id: ai-link-routing
title: Routing, Retries & Timeouts
sidebar_label: Routing & Retries
---

# Routing, Retries & Timeouts

How the Hub decides which upstream call to make, when it tries again, and how long it waits.

## Composite targets

A composite `aiTargets` entry fans a single `x-target` id out to several concrete targets:

```yaml
aiTargets:
  - id: prod-chat
    strategy: fallback            # or: loadbalance
    routes:
      - target: openai
      - target: anthropic
        weight: 1                 # weights apply to loadbalance only
```

| Field            | Required | Description                                                            |
| ---------------- | -------- | ---------------------------------------------------------------------- |
| `id`             | yes      | Value of the `x-target` header; takes priority over a legacy target of the same id |
| `strategy`       | yes      | `fallback` or `loadbalance`                                            |
| `routes`         | yes      | Non-empty list of `{ target: <concrete aiTarget id>, weight?: number }` |
| `requestTimeout` | no       | Inherited by any route that has no `requestTimeout` of its own          |
| `debug`          | no       | Applied to every route it executes, unless the route overrides the flag |

Validation rules, enforced at startup:

- `retry:` next to `strategy:`/`routes:` is rejected — attach it to the routes (`aiTargets composite '<id>' cannot declare retry — attach it to its routes instead`).
- `guardrails:` next to `strategy:`/`routes:` is rejected — attach them to the routes (`aiTargets composite '<id>' cannot declare guardrails — attach them to its routes`).
- Routes must reference an existing concrete target. Unknown ids and nested composites are rejected.
- `id` must be unique across `aiTargets`.

### `fallback`

Routes are attempted **in configuration order**:

```yaml
aiTargets:
  - id: prod-chat
    strategy: fallback
    routes:
      - target: openai
      - target: azure-openai-chat
      - target: anthropic
```

### `loadbalance`

The first route is picked **weighted-randomly**; the remaining routes follow in descending weight order (equal weights keep configuration order) so a failed pick still has an ordered fallback chain:

```yaml
aiTargets:
  - id: prod-chat
    strategy: loadbalance
    routes:
      - target: openai
        weight: 3
      - target: anthropic
        weight: 1
```

`weight` must be greater than 0 and defaults to `1`.

## When a request moves to the next route

After a route exhausts its own retries, the Hub moves to the next route **only if**:

- the failure is retryable — HTTP `429`, HTTP `5xx`, a timeout, or a network error (`ECONNREFUSED`, `ECONNRESET`, `ETIMEDOUT`, `ECONNABORTED`, `ENOTFOUND`, `EAI_AGAIN`), and
- nothing has been streamed to the client yet, and
- there is another route left.

Non-retryable failures — `400`, `401`, `403`, `422`, and any other 4xx — stop the chain immediately, even if later routes are configured: a bad API key or a missing model will not be fixed by another provider. A client disconnect is never retried and never falls back.

Composite routing applies to chat completions and the Responses API. Image generation, audio speech, audio transcription and audio translation make a single attempt against the requested target and ignore composites.

## Retry and backoff

Retries are **owned by the AI engine** itself — there is no hidden provider-level retry underneath — so they compose predictably with composite fallback and telemetry.

Global defaults (`hub.http_client.retry`):

```yaml
hub:
  http_client:
    retry:
      maxAttempts: 3          # 1 initial attempt + 2 retries
      initialDelayMs: 500
      maxDelayMs: 5000
      backoffMultiplier: 2
```

Per-target override — fields you omit inherit the global value:

```yaml
aiTargets:
  - id: flaky-provider
    provider: openai
    apiKey: ${OPENAI_API_KEY}
    retry:
      maxAttempts: 5
      initialDelayMs: 1000
    options:
      model: gpt-6-luna

  - id: single-shot
    provider: anthropic
    apiKey: ${ANTHROPIC_API_KEY}
    retry:
      maxAttempts: 1          # disables retries for this target
    options:
      model: claude-sonnet-5-5
```

Backoff is **equal-jitter exponential**: delay `n` is 50–100 % of `min(maxDelayMs, initialDelayMs × backoffMultiplier^n)`. The jitter avoids a thundering herd against a shared upstream outage.

Retry scope: chat completions and the Responses API, including their streaming paths.

## Timeouts

Timeout precedence for a single route:

1. the route's own `requestTimeout`
2. the composite's `requestTimeout` (when the requested id is a composite)
3. `hub.http_client.timeout` (default `120000`)

`requestTimeout: 0` disables the timeout entirely for that target.

Two behaviours to plan for:

- **All retries of one route share that route's timeout budget.** The timer is armed once per route, not per attempt — retrying does not grant fresh headroom. A composite's `requestTimeout` is inherited per route; it does not span the whole chain.
- **A live stream's timeout covers time-to-first-byte only**, not the whole stream — unless a blocking post-phase guardrail forces internal buffering, in which case it covers the entire generation (see [Guardrails](./ai-link-guardrails.md#streaming-behaviour)).

:::tip[Raising attempts and timeouts together]
If you raise `maxAttempts` (or the backoff delays) on a target whose upstream is slow, raise `requestTimeout` as well. With a shared budget, a request that would have finished inside 120 s across 3 attempts can now exceed it.
:::

## What a fallback chain looks like

`prod-chat` is a `fallback` composite over routes A (`openai`) and B (`anthropic`), with the default retry policy (`maxAttempts: 3`):

1. Route A, attempt 1 → upstream answers `429`. Retryable, attempts remain → backoff, retry.
2. Route A, attempt 2 → upstream answers `503`. Retryable → backoff, retry.
3. Route A, attempt 3 → upstream answers `503`. Retryable, but this route's attempts are exhausted → the Hub logs that route A failed retryably and moves to route B.
4. Route B, attempt 1 → `200`. The client receives route B's response.

Nothing has been streamed to the client at any point, which is what makes steps 1–3 invisible to it. If route B had also failed retryably, the client would get a `502` describing the last failure.

## Observability

Each composite attempt is recorded individually in the runtime usage tracker: one error record per failed attempt, and `recordSuccess` (or a terminal `recordError`) on the last executed target. Consequently a composite that retries N times inflates the AI-Link request/error totals by N−1 compared with what a single logical request would otherwise count.

That inflation is also tracked explicitly as **fallback attempts**, shown in the [Admin Panel](../admin-panel.md) AI-Link tab, next to the per-target tables of requests, errors and latency. Deprecation warnings are separate: they appear as `aiLink` log lines whenever a request is served by a legacy target.
