---
id: ai-link-migration
title: Migrating from targets to aiTargets
sidebar_label: Migration to aiTargets
---

# Migrating from targets to aiTargets

Hub 1.3.0 moves AI-Link to a new, built-in AI engine. Targets declared under `aiTargets:` run on it; the old `targets:` section still works through the legacy AI gateway, but is deprecated.

:::danger[Action required in Hub 1.3.0]
Legacy `targets:` entries are **deprecated** and will be removed in a future release.

- Every request served by a legacy target logs: `AI-Link request served via deprecated legacy target — migrate '<id>' to aiTargets` (log type `aiLink`, level `warn`).
- New capabilities exist only on `aiTargets`: composite routing, guardrails, engine-owned retries, the Responses API, the Realtime relay, and upstream wire logging.
- Migration is per-target and transparent to clients: an `aiTargets` entry with the **same id** takes over the same `x-target` value.
:::

## Why Migrate

- **No separate gateway process.** The Hub serves `aiTargets` with its new built-in AI engine. Once no legacy targets remain, the legacy AI gateway is not started at all: one less process, one less port, less memory and CPU, and `/ready` no longer depends on it.
- **Engine-owned retries.** Retryable upstream failures (429, 5xx, timeouts, network errors) are retried with backoff by default on chat completions and the Responses API. See [Routing & Retries](./ai-link-routing.md).
- **Composite routing.** `fallback` and weighted `loadbalance` across several targets under a single `x-target` id.
- **Guardrails.** Regex, length, JSON, PII, LLM-judge and moderation-API checks with `deny` / `redact` / `log` actions, applied before the request, after the response, or both. See [Guardrails](./ai-link-guardrails.md).
- **`POST /v1/responses`.** The Responses API is available on `aiTargets` only.
- **Realtime relay.** Voice-agent WebSocket sessions require a concrete OpenAI `aiTargets` entry. See [Realtime Relay](./realtime.md).
- **Provider-native authentication.** Azure OpenAI through Microsoft Entra ID, AWS Bedrock through STS role assumption or the default credential chain, Vertex AI with automatic private-key normalization.
- **Upstream wire logging.** `logUpstream` records the provider-native request/response bodies (and the upstream error body) for `aiTargets`.

## Migration Procedure

1. **Inventory the legacy ids.** Every legacy request logs a deprecation warning naming its id; the [Admin Panel](../admin-panel.md) AI-Link tab also lists the configured targets. Also check (access logs or client owners) whether any id is called on `/v1/embeddings`, `/v1/completions` or `/v1/rerank` — those must stay legacy, see [Not migratable](#other-providers).
2. **Add an `aiTargets` entry per legacy target, keeping the same `id`.** `aiTargets` wins when an id exists in both lists, so ProActions YAML (`SERVICES.*.target:` or a per-step `target:`) needs no change, and the legacy entry stays in place as a dormant rollback.
3. **Translate the fields** using the [general mapping](#general-field-mapping) and the provider sections below.
4. **Restart the Hub.** Schema errors abort startup and name the offending key (for example `"awsRegion" is not allowed`).
5. **Test each target** with curl, including `"stream": true`.
6. **Confirm no deprecation warnings** appear for that id in the `aiLink` logs.
7. **Delete the legacy entries.** Once `targets:` is empty or removed, the legacy AI gateway is not started and `/ready` reports its `ai_gateway` check as `skipped: true`. Restart to apply.
8. **Optionally remove the `hub.ailink_*` overrides** from your environment (see [Legacy targets](./ai-link-legacy.md#legacy-gateway-settings)).

:::tip[Rollback]
Delete the `aiTargets` entry (or rename its `id`) and the legacy entry with the same id serves again. The Gateway is only started when at least one legacy target exists **at startup**, so a restart is required.
:::

## General Field Mapping

| Legacy (`targets:`)                          | New (`aiTargets:`)                                        |
| -------------------------------------------- | --------------------------------------------------------- |
| `id`                                         | `id` — keep it identical so clients need no change         |
| `provider`                                   | `provider` — see the [provider names](#provider-names)     |
| `apiKey`                                     | `apiKey`                                                  |
| `options` (including `options.model`)        | `options` — unchanged, OpenAI wire names                   |
| `requestTimeout`                             | `requestTimeout`                                          |
| `headers`                                    | `settings.headers`                                        |
| `openaiOrganization` / `openaiProject`       | `settings.organization` / `settings.project`              |
| `azureResourceName` / `azureApiVersion`      | `settings.resourceName` / `settings.apiVersion`            |
| `azureDeploymentId`                          | `options.model`                                           |
| `awsAccessKeyId` / `awsSecretAccessKey`      | `settings.accessKeyId` / `settings.secretAccessKey`        |
| `awsSessionToken` / `awsRegion`              | `settings.sessionToken` / `settings.region`                |
| `vertexProjectId` / `vertexRegion`           | `settings.project` / `settings.location`                   |
| `clientEmail` / `privateKey`                 | `settings.googleAuthOptions.credentials.client_email` / `.private_key` |
| `scope`                                      | drop (default `https://www.googleapis.com/auth/cloud-platform`), or `settings.googleAuthOptions.scopes` |
| `endpoint: https://host/base{path}` + `provider: custom` | `provider: openai-compatible` + `settings.baseURL: https://host/base/v1` |
| `config:` (legacy routing object)            | a composite `aiTargets` entry (`strategy` + `routes`)      |
| `config.beforeRequestHooks` guardrails        | top-level `guardrails:` entries, attached via `guardrails: [name]` |

The `/v1` moves into `baseURL` because the legacy `{path}` placeholder included it: `https://api.writer.com{path}` becomes `https://api.writer.com/v1`. Do not include `/chat/completions` (or any other endpoint path) in `baseURL` — the engine appends it.

### Provider names

| Legacy provider | `aiTargets` provider | Notes                                        |
| --------------- | -------------------- | -------------------------------------------- |
| `azure-openai`  | `azure`              | `azure-openai` is still accepted as an alias  |
| `vertex-ai`     | `vertex`             | `vertex-ai` and `google-vertex-ai` are aliases |
| `google`        | `google`             | Aliases: `gemini`, `google-ai`                |
| `bedrock`       | `bedrock`            | Alias: `amazon-bedrock`                       |
| `perplexity-ai` | `perplexity`         | **Rename required**                           |
| `mistral-ai`    | `mistral`            | **Rename required**                           |
| `together-ai`   | `togetherai`         | **Rename required**                           |
| `fireworks-ai`  | `fireworks`          | **Rename required**                           |
| `x-ai`          | `xai`                | **Rename required**                           |

Canonical `aiTargets` providers: `openai`, `azure`, `anthropic`, `google`, `vertex`, `bedrock`, `mistral`, `groq`, `deepseek`, `xai`, `cohere`, `togetherai`, `fireworks`, `perplexity`, `huggingface`, `openrouter`, `openai-compatible`.

## OpenAI

| Legacy                     | New                      |
| -------------------------- | ------------------------ |
| `apiKey`                   | `apiKey`                 |
| `openaiOrganization`       | `settings.organization`  |
| `openaiProject`            | `settings.project`       |
| `options`                  | `options` (unchanged)    |

Before:

```yaml
targets:
  - id: openai
    provider: openai
    apiKey: ${OPENAI_API_KEY}
    options:
      model: gpt-6-luna
      max_completion_tokens: 800
```

After:

```yaml
aiTargets:
  - id: openai
    provider: openai
    apiKey: ${OPENAI_API_KEY}
    options:
      model: gpt-6-luna
      max_completion_tokens: 800
```

:::caution[Unknown body parameters are dropped]
The legacy path forwarded arbitrary body keys to the provider. `aiTargets` forward only the documented OpenAI parameters; anything else is dropped and logged as `AI-Engine dropped unsupported params: <names>`. Use the target's `providerOptions.openai` for OpenAI-specific extras.
:::

## Anthropic

| Legacy      | New       |
| ----------- | --------- |
| `apiKey`    | `apiKey`  |
| `options`   | `options` |

Before:

```yaml
targets:
  - id: anthropic
    provider: anthropic
    apiKey: ${ANTHROPIC_API_KEY}
    options:
      model: claude-sonnet-5-5
      max_tokens: 800
```

After:

```yaml
aiTargets:
  - id: anthropic
    provider: anthropic
    apiKey: ${ANTHROPIC_API_KEY}
    options:
      model: claude-sonnet-5-5
      max_tokens: 800
```

Keep `max_tokens` in `options` — Anthropic requires it. Provider-specific features such as thinking `effort` go through `providerOptions.anthropic` (see [Request defaults and precedence](./ai-link.md#request-defaults-and-precedence)).

## AWS Bedrock

| Legacy                | New                        |
| --------------------- | -------------------------- |
| `awsAccessKeyId`      | `settings.accessKeyId`     |
| `awsSecretAccessKey`  | `settings.secretAccessKey` |
| `awsSessionToken`     | `settings.sessionToken`    |
| `awsRegion`           | `settings.region`          |
| `apiKey`              | *(not used)*               |

Before:

```yaml
targets:
  - id: bedrock
    provider: bedrock
    awsAccessKeyId: ${AWS_BEDROCK_ACCESS_KEY_ID}
    awsSecretAccessKey: ${AWS_BEDROCK_SECRET_ACCESS_KEY}
    awsRegion: ${AWS_BEDROCK_REGION}
    options:
      model: "eu.anthropic.claude-haiku-5-5"
      max_tokens: 800
```

After:

```yaml
aiTargets:
  - id: bedrock
    provider: bedrock
    settings:
      region: ${AWS_BEDROCK_REGION}
      accessKeyId: ${AWS_BEDROCK_ACCESS_KEY_ID}
      secretAccessKey: ${AWS_BEDROCK_SECRET_ACCESS_KEY}
    options:
      model: "eu.anthropic.claude-haiku-5-5"
      max_tokens: 800
```

### Inference profiles / ARNs

`options.model` accepts a cross-region inference-profile id or a full ARN exactly as before. Long-context models keep their raised timeout:

```yaml
aiTargets:
  - id: bedrock-nda
    provider: bedrock
    settings:
      region: ${AWS_BEDROCK_REGION_NDA}
      accessKeyId: ${AWS_BEDROCK_ACCESS_KEY_ID_NDA}
      secretAccessKey: ${AWS_BEDROCK_SECRET_ACCESS_KEY_NDA}
    requestTimeout: 300000   # 5 minutes for long-context models
    options:
      model: "arn:aws:bedrock:eu-north-1:123456789012:inference-profile/eu.anthropic.claude-sonnet-5-5"
      max_tokens: 8000
```

### New: IAM role assumption

Legacy targets could not assume a role. On `aiTargets`, omit the static keys and set `roleArn`:

```yaml
aiTargets:
  - id: bedrock-role
    provider: bedrock
    settings:
      region: ${AWS_BEDROCK_REGION}
      roleArn: ${AWS_BEDROCK_ROLE_ARN}
      externalId: ${AWS_BEDROCK_EXTERNAL_ID}   # optional, cross-account roles
    options:
      model: "eu.anthropic.claude-haiku-5-5"
      max_tokens: 800
```

The STS call uses the target's default AWS credential chain; the assumed credentials are cached in memory and refreshed about 60 s before expiry.

### New: default credential chain

Omit `accessKeyId`, `secretAccessKey` and `roleArn` to use whatever the host provides — instance/container role, environment variables, shared config or SSO:

```yaml
aiTargets:
  - id: bedrock-default-chain
    provider: bedrock
    settings:
      region: ${AWS_BEDROCK_REGION}
    options:
      model: "eu.anthropic.claude-haiku-5-5"
      max_tokens: 800
```

Credential precedence: static keys (both set) → `roleArn` STS assumption → default chain.

## Google

Legacy `provider: google` (Gemini API with an API key) and `provider: vertex-ai` (GCP service account) map to **two different** `aiTargets` providers.

### Google Gemini API

| Legacy            | New              |
| ----------------- | ---------------- |
| `provider: google` | `provider: google` (aliases `gemini`, `google-ai`) |
| `apiKey`          | `apiKey`         |
| `options`         | `options`        |

Before:

```yaml
targets:
  - id: gemini
    provider: google
    apiKey: ${GEMINI_API_KEY}
    options:
      model: gemini-3.8-flash
```

After:

```yaml
aiTargets:
  - id: gemini
    provider: google
    apiKey: ${GEMINI_API_KEY}
    options:
      model: gemini-3.8-flash
```

### Google Vertex AI

| Legacy              | New                                                            |
| ------------------- | -------------------------------------------------------------- |
| `provider: vertex-ai` | `provider: vertex` (alias `vertex-ai` still accepted)          |
| `vertexProjectId`   | `settings.project`                                             |
| `vertexRegion`      | `settings.location`                                            |
| `clientEmail`       | `settings.googleAuthOptions.credentials.client_email`           |
| `privateKey`        | `settings.googleAuthOptions.credentials.private_key`            |
| `scope`             | drop it — the default is `https://www.googleapis.com/auth/cloud-platform`; for a custom scope use `settings.googleAuthOptions.scopes: [<scope>]` |
| `apiKey`            | *(not used)*                                                    |

Before:

```yaml
targets:
  - id: vertex
    provider: vertex-ai
    privateKey: ${VERTEX_PRIVATE_KEY}
    clientEmail: ${VERTEX_CLIENT_EMAIL}
    scope: ${VERTEX_SCOPE}
    vertexProjectId: ${VERTEX_PROJECT_ID}
    vertexRegion: ${VERTEX_REGION}
    options:
      model: gemini-3.8-flash
```

After:

```yaml
aiTargets:
  - id: vertex
    provider: vertex
    settings:
      project: ${VERTEX_PROJECT_ID}
      location: ${VERTEX_REGION}
      googleAuthOptions:
        credentials:
          client_email: ${VERTEX_CLIENT_EMAIL}
          private_key: ${VERTEX_PRIVATE_KEY}
    options:
      model: gemini-3.8-flash
```

:::tip[Private key formatting]
An environment variable holding the PEM key with literal `\n` sequences works as-is — the Hub normalizes them to real newlines. `DECODER routines::unsupported` at runtime means the key content itself is malformed, not that the quoting style is wrong.
:::

## Other Providers

| Provider        | Legacy                                              | New                                                                    |
| --------------- | --------------------------------------------------- | ---------------------------------------------------------------------- |
| Azure OpenAI    | `provider: azure-openai`, `azureResourceName`, `azureApiVersion`, `azureDeploymentId` | `provider: azure`, `settings.resourceName`, `settings.apiVersion`, `options.model`; add `settings.useEntraId: true` for Entra ID |
| Mistral         | `provider: mistral-ai`, `apiKey`                    | `provider: mistral`, `apiKey`                                          |
| OpenRouter      | `provider: openrouter`, `apiKey`                    | unchanged                                                              |
| Perplexity      | `provider: perplexity-ai`, `apiKey`                 | `provider: perplexity`, `apiKey`                                       |
| Groq            | `provider: groq`, `apiKey`                          | unchanged                                                              |
| DeepSeek        | `provider: deepseek`, `apiKey`                      | unchanged                                                              |
| xAI             | `provider: x-ai`, `apiKey`                          | `provider: xai`, `apiKey`                                              |
| Together        | `provider: together-ai`, `apiKey`                   | `provider: togetherai`, `apiKey`                                       |
| Fireworks       | `provider: fireworks-ai`, `apiKey`                  | `provider: fireworks`, `apiKey`                                        |
| Hugging Face    | `provider: huggingface`, `apiKey`                   | unchanged                                                              |
| Ollama          | `provider: ollama`                                  | `provider: openai-compatible`, `settings.baseURL: http://<host>:11434/v1` |
| Any custom OpenAI-wire endpoint | `provider: custom`, `endpoint: https://host/base{path}` | `provider: openai-compatible`, `settings.baseURL: https://host/base/v1` |

:::caution[Not migratable]
Some legacy capabilities have no `aiTargets` equivalent and must stay on a legacy target for now:

- Targets called on `/v1/embeddings`, `/v1/completions` or `/v1/rerank`. The new engine does not serve these endpoints: once an `aiTargets` entry takes over the id, those calls return `400` `Endpoint <path> is not supported for aiTargets`. Keep such an id on its legacy target — including when the same id also serves chat.
- Providers with neither a native `aiTargets` provider nor an OpenAI-compatible endpoint (for example Cloudflare Workers AI and Amazon SageMaker).
- Legacy-only request features, such as Langdock assistants (`provider: custom` with `options.assistantId`).

Report these to Eidosmedia support before the removal release.
:::

## Migrating Load Balancing and Fallback (`config:`)

A legacy `config:` routing object becomes a composite `aiTargets` entry. Each entry of its `targets[]` list becomes a standalone `aiTargets` entry of its own, and the composite references them by id.

Before:

```yaml
targets:
  - id: custom_config
    config:
      strategy:
        mode: loadbalance
      targets:
        - provider: openai
          api_key: ${OPENAI_API_KEY}
          override_params:
            model: gpt-6-luna
        - provider: openrouter
          api_key: ${OPENROUTER_API_KEY}
          override_params:
            model: anthropic/claude-sonnet-5.5
```

After:

```yaml
aiTargets:
  - id: custom_config-a
    provider: openai
    apiKey: ${OPENAI_API_KEY}
    options:
      model: gpt-6-luna

  - id: custom_config-b
    provider: openrouter
    apiKey: ${OPENROUTER_API_KEY}
    options:
      model: anthropic/claude-sonnet-5.5

  - id: custom_config
    strategy: loadbalance
    routes:
      - target: custom_config-a
      - target: custom_config-b
```

:::note[No inline per-route overrides]
Composite routes cannot carry `override_params` — they reference a concrete target by id. Set `options` on the route targets instead, as above. Weights are expressed with `routes[].weight` (`loadbalance` only).
:::

See [Routing, Retries & Timeouts](./ai-link-routing.md) for `fallback` vs `loadbalance` semantics and validation rules.

## Migrating Guardrail Hooks

Legacy `config.beforeRequestHooks` become entries in the top-level `guardrails:` list, attached to a target by name.

Before:

```yaml
targets:
  - id: custom_config_guardrails
    config:
      provider: openai
      api_key: ${OPENAI_API_KEY}
      override_params:
        model: gpt-6-luna
      beforeRequestHooks:
        - type: guardrail
          id: mistral_check
          deny: true
          checks:
            - id: mistral.moderateContent
              parameters:
                credentials:
                  apiKey: ${MISTRAL_API_KEY}
                categories:
                  - pii
                  - sexual
                  - hate_and_discrimination
```

After:

```yaml
aiTargets:
  - id: custom_config_guardrails
    provider: openai
    apiKey: ${OPENAI_API_KEY}
    options:
      model: gpt-6-luna
    guardrails: [custom_config_guardrails-moderation]

  # Dedicated moderation-only target. runModeration() falls back to OpenAI's
  # default baseURL when settings.baseURL is unset, so it must be explicit here.
  - id: mistral-moderation
    provider: mistral
    apiKey: ${MISTRAL_API_KEY}
    settings:
      baseURL: https://api.mistral.ai/v1

guardrails:
  - name: custom_config_guardrails-moderation
    type: llm
    mode: moderation-api
    target: mistral-moderation
    model: mistral-moderation-latest
    categories: [pii, sexual, hate_and_discrimination]
    threshold: 0.8
    onFail: deny
    onError: allow
```

:::warning[Fail-open choice]
`onError: allow` means a moderation-endpoint outage silently disables enforcement on that target. Use `onError: deny` when the check must be fail-closed.
:::

Full reference, including `pre`/`post`/`both` phases, `redact` and `log` actions, and streaming behaviour: [Guardrails](./ai-link-guardrails.md).

## Behavior Differences to Expect

- **Retries are on by default.** Chat completions and the Responses API use 3 total attempts with backoff. Set `retry: { maxAttempts: 1 }` on a target to keep the legacy single-shot behaviour.
- **Unknown request parameters are dropped** and logged, instead of being forwarded to the provider.
- **Error bodies have a fixed shape.** Upstream failures return the `502` JSON documented in [AI-Link errors](./ai-link.md#errors) — not the legacy gateway's error envelope.
- **Streaming failures before the first byte return `502`.** The legacy path could deliver a partially-committed stream; the new path detects a pre-header failure and can retry or fall back.
- **`debug` and `logUpstream` apply to `aiTargets` only.** Legacy targets ignore them.
- **`hub.ailink_*` settings do not affect `aiTargets`.** They only configure the legacy AI gateway subprocess.
- **Images and audio make a single attempt.** Retries and composite fallback cover chat completions and the Responses API only.

## Migration Checklist

- [ ] Legacy target ids inventoried (deprecation warnings, Admin Panel AI-Link tab)
- [ ] No id to be migrated is used on `/v1/embeddings`, `/v1/completions` or `/v1/rerank`
- [ ] `aiTargets` entry created for each legacy target, **with the same `id`**
- [ ] Provider names canonicalized (`perplexity-ai` → `perplexity`, `mistral-ai` → `mistral`, `together-ai` → `togetherai`, `fireworks-ai` → `fireworks`, `x-ai` → `xai`)
- [ ] Provider-specific fields moved under `settings:`
- [ ] `options.model` set on every concrete target
- [ ] Custom endpoints converted to `provider: openai-compatible` + `settings.baseURL` (with `/v1`)
- [ ] Legacy `config:` routing objects converted to composite entries
- [ ] Legacy `beforeRequestHooks` converted to `guardrails:` entries and attached
- [ ] `retry` / `requestTimeout` reviewed (timeouts shared across a route's retries)
- [ ] Hub restarted; startup schema validation passes
- [ ] Each target tested with curl, including `stream: true`
- [ ] No `deprecated legacy target` warnings remain for migrated ids
- [ ] Legacy entries deleted; `/ready` shows the `ai_gateway` check `skipped: true`
- [ ] Unused `hub.ailink_*` environment overrides removed
