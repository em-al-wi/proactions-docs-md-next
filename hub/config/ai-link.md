---
id: ai-link
title: AI-Link Configuration
sidebar_label: AI-Link
sidebar_position: 3
---

# AI-Link Configuration

The AI-Link module provides a unified, OpenAI-compatible gateway to multiple AI service providers. Providers are declared as **AI-Link targets** (`aiTargets`) and served by the Hub's new built-in AI engine — no separate gateway process, with native provider authentication, automatic retries, composite routing and guardrails.

:::warning[Legacy targets are deprecated]
Provider entries declared under `targets:` are served by the legacy AI gateway, a separate subprocess the Hub starts for them. They are deprecated since Hub 1.3.0: they keep working in 1.3.x, but every request logs a deprecation warning, and support will be removed in a future release. See [Legacy targets](./ai-link-legacy.md) for the old format and [Migrating from targets to aiTargets](./ai-link-migration.md) for the conversion.
:::

## Quick Start

### Minimal Configuration

The simplest configuration requires an `id`, a `provider`, and an API key:

```yaml
aiTargets:
  - id: openai
    provider: openai
    apiKey: ${OPENAI_API_KEY}
    options:
      model: gpt-6-luna
```

Set the environment variable:

```bash
OPENAI_API_KEY=sk-your-api-key-here
```

### Using the Target

Make requests to AI-Link endpoints with the `x-target` header:

```bash
curl -X POST http://localhost:$PROACTIONS_HUB_PORT/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "x-target: openai" \
  -d '{
    "messages": [
      {"role": "user", "content": "Hello!"}
    ]
  }'
```

The `model` from the target's `options` is merged into the request body. A `model` in the request body overrides it.

## How Requests Are Routed

The `x-target` header (or a `?target=` query parameter) selects the target. Resolution order is:

1. The id matches an entry in `aiTargets` → the Hub serves the request in-process through the matching provider. If the entry is a composite (`strategy` + `routes`), the Hub resolves the attempt order first — see [Routing & Retries](./ai-link-routing.md).
2. Otherwise the id matches an entry in the legacy `targets` list → the request goes through the legacy AI gateway, and the Hub logs one deprecation warning for that id.
3. Otherwise the Hub returns `400` with `message: "INVALID_TARGET"` and `error: "No target with name <id> found"` (when no legacy `targets` are configured at all, the error reads `No targets configured with baseType targets`). A request without `x-target` or `?target=` returns `400` `MISSING_TARGET`.

If the same id exists in both lists, **the `aiTargets` entry wins** — this is what makes migration transparent. Clients, and ProActions YAML `HUB` service `target:` / `x-target` values, need no change when a legacy target is migrated to an `aiTargets` entry with the same id.

## Available Endpoints

AI-Link provides OpenAI-compatible endpoints:

- **Chat Completions:** `POST /v1/chat/completions`
- **Image Generation:** `POST /v1/images/generations`
- **Audio Speech:** `POST /v1/audio/speech`
- **Audio Transcription:** `POST /v1/audio/transcriptions`
- **Audio Translation:** `POST /v1/audio/translations`
- **Responses API:** `POST /v1/responses`

All six are served by `aiTargets`. The Responses API is available on `aiTargets` only — there is no legacy fallback for it.

### Modality support

Not every provider supports every modality. An unsupported provider/modality combination returns `400` with the code `UNSUPPORTED_MODALITY` rather than a raw provider error — for example image generation on Anthropic or Mistral.

Image generation forwards `n`, `size`, `quality`, `output_format` and `background` to OpenAI and Azure image models; an invalid value for the last three returns `400` `INVALID_IMAGE_OPTION`.

Audio translation is available on `openai` and `openai-compatible` targets only, through the providers' OpenAI-compatible file-translation endpoint; other providers return `UNSUPPORTED_MODALITY` instead of silently performing same-language transcription. Accepted `response_format` values for translation are `json`, `text`, and `verbose_json`.

### Responses API subset

Supported:

- Input as a plain string, or message items with roles `user`, `system`, `developer`, `assistant` and `input_text` (or string) content
- Function tools and `tool_choice`
- Streaming (SSE)
- `instructions`
- `text.format` (structured output)
- `reasoning.effort` (same values as `reasoning_effort`)

Rejected with `400`: `previous_response_id` (send the full input on every request) and built-in tools (`web_search`, `file_search`, `code_interpreter`).

### Legacy-only endpoints

`POST /v1/embeddings`, `POST /v1/completions` and `POST /v1/rerank` are served only by legacy targets. Called with an `aiTargets` id, they return `400` with `Endpoint <path> is not supported for aiTargets`.

### Realtime

These HTTP endpoints do not provide OpenAI Realtime sessions. `HUB_VOICE_AGENT` uses the dedicated WebSocket relay instead — see [Realtime Relay](./realtime.md).

## Streaming

The Hub natively proxies Server-Sent Events (SSE) streaming responses. Set `"stream": true` in the request body and the Hub forwards the upstream SSE stream directly to the client.

```bash
curl -X POST http://localhost:${PROACTIONS_HUB_PORT}/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "x-target: openai" \
  --no-buffer \
  -d '{
    "messages": [{"role": "user", "content": "Tell me a short story"}],
    "stream": true
  }'
```

The response uses `Content-Type: text/event-stream` and follows the standard OpenAI streaming format — each event is a `data:` line containing a JSON chunk, terminated by `data: [DONE]`.

**Behaviour:**

- Client disconnects are detected immediately — the upstream connection is torn down to prevent resource leaks. A client disconnect is never retried.
- Streaming requests appear separately in the [Admin Panel](../admin-panel.md) AI-Link metrics (streaming ratio).
- A failure **before the first byte reaches the client** (connect error, `429`, `5xx`, timeout) is retried and can fall back to the next composite route; if every attempt fails, the client gets a clean `502` response and no partial stream.
- A failure **after the headers are sent** closes the stream — bytes already delivered cannot be un-sent.

## Target Configuration Structure

Each `aiTargets` entry:

| Field             | Required | Description                                                                                                        |
| ----------------- | -------- | ------------------------------------------------------------------------------------------------------------------ |
| `id`              | yes      | Identifier sent in the `x-target` header; unique across `aiTargets`                                                 |
| `provider`        | yes      | Provider name (see [Supported Providers](#supported-providers))                                                     |
| `apiKey`          | no       | Provider credential (API key)                                                                                        |
| `settings`        | no       | Provider connection settings (`baseURL`, `headers`, `region`, …), passed to the provider client as-is                |
| `options`         | no       | Default request-body parameters, using OpenAI wire names. **The model belongs here: `options.model`**                |
| `providerOptions` | no       | Provider-specific options keyed by provider id (e.g. `anthropic`), forwarded verbatim on chat, Responses, image, speech and transcription calls |
| `requestTimeout`  | no       | Timeout in milliseconds; `0` disables the timeout. Default: `hub.http_client.timeout` (`120000`)                    |
| `retry`           | no       | Per-target retry policy — see [Routing & Retries](./ai-link-routing.md)                                             |
| `debug`           | no       | Per-target `logPrompt` / `logResponse` / `logUpstream` overrides — see [Logging](./logging.md)                      |
| `guardrails`      | no       | Names of entries in the top-level `guardrails:` list — see [Guardrails](./ai-link-guardrails.md)                    |
| `realtime`        | no       | `enabled`, `maxSessionDurationMs`, `idleTimeoutMs`, `maxMessageBytes`, `maxBytesPerSecond` — see [Realtime Relay](./realtime.md) |

:::warning[No unknown keys]
`aiTargets` entries reject unknown keys. Provider-specific settings (`region`, `headers`, `baseURL`, AWS keys, Azure resource names, …) must live under `settings:`, never at the entry root. Startup fails with `"<key>" is not allowed` naming the offending key.
:::

```yaml
aiTargets:
  - id: example
    provider: openai
    apiKey: ${EXAMPLE_API_KEY}
    settings:
      headers:
        X-Custom-Header: value
    options:
      model: gpt-6-luna
      max_completion_tokens: 2000
    providerOptions: {}
    requestTimeout: 120000
    retry:
      maxAttempts: 3
      initialDelayMs: 500
      maxDelayMs: 5000
      backoffMultiplier: 2
    debug:
      logUpstream: false
    guardrails: []
```

### Composite targets

A composite entry fans one `x-target` id out to several concrete targets:

```yaml
aiTargets:
  - id: prod-chat
    strategy: fallback        # or: loadbalance
    routes:
      - target: openai
      - target: anthropic
        weight: 1             # weights apply to loadbalance only
```

Composites may not declare `retry:` or `guardrails:` directly (both are rejected with an id-specific error — attach them to the routes instead), may not nest other composites, and may not reference unknown ids. See [Routing & Retries](./ai-link-routing.md).

## Supported Providers

| Provider id        | Aliases                        | Authentication                    | Common `settings`                                            |
| ------------------ | ------------------------------ | --------------------------------- | ------------------------------------------------------------ |
| `openai`           | —                              | `apiKey`                          | `baseURL`, `headers`, `organization`, `project`              |
| `azure`            | `azure-openai`                 | `apiKey` or Entra ID              | `resourceName`, `apiVersion`, `baseURL`, `headers`, `useEntraId`, `entraScope` |
| `anthropic`        | —                              | `apiKey`                          | `baseURL`, `headers`, `authToken`                            |
| `google`           | `gemini`, `google-ai`          | `apiKey`                          | `baseURL`, `headers`                                         |
| `vertex`           | `vertex-ai`, `google-vertex-ai`| Service account                   | `project`, `location`, `googleAuthOptions`, `baseURL`, `headers` |
| `bedrock`          | `amazon-bedrock`               | AWS keys, IAM role, or default chain | `region`, `accessKeyId`, `secretAccessKey`, `sessionToken`, `roleArn`, `externalId` |
| `mistral`          | —                              | `apiKey`                          | `baseURL`, `headers`                                         |
| `groq`             | —                              | `apiKey`                          | `baseURL`, `headers`                                         |
| `deepseek`         | —                              | `apiKey`                          | `baseURL`, `headers`                                         |
| `xai`              | —                              | `apiKey`                          | `baseURL`, `headers`                                         |
| `cohere`           | —                              | `apiKey`                          | `baseURL`, `headers`                                         |
| `togetherai`       | —                              | `apiKey`                          | `baseURL`, `headers`                                         |
| `fireworks`        | —                              | `apiKey`                          | `baseURL`, `headers`                                         |
| `perplexity`       | —                              | `apiKey`                          | `baseURL`, `headers`                                         |
| `huggingface`      | —                              | `apiKey`                          | `baseURL`, `headers`                                         |
| `openrouter`       | —                              | `apiKey`                          | `baseURL`, `headers`                                         |
| `openai-compatible`| —                              | `apiKey`                          | `baseURL` (required in practice), `headers`                  |

Credentials and settings left unset fall back to the provider's environment variable where one exists: `OPENAI_API_KEY`, `ANTHROPIC_API_KEY` and `GOOGLE_GENERATIVE_AI_API_KEY` for the API key; `AWS_REGION` for Bedrock `settings.region`; `GOOGLE_VERTEX_PROJECT` / `GOOGLE_VERTEX_LOCATION` for Vertex `settings.project` / `settings.location`.

:::info[Any other provider]
Any service exposing an OpenAI-compatible endpoint can be used through `openai-compatible` with `settings.baseURL` — Writer, Langdock, Ollama, Azure AI Inference, and similar. If a service has neither a native provider nor an OpenAI-compatible endpoint, keep it on a [legacy target](./ai-link-legacy.md) for now and contact Eidosmedia support.
:::

## Provider Examples

### OpenAI

```yaml
aiTargets:
  - id: openai
    provider: openai
    apiKey: ${OPENAI_API_KEY}
    settings:
      organization: ${OPENAI_ORG}   # Optional
      project: ${OPENAI_PROJECT}     # Optional
    options:
      model: gpt-6.1-sol
      max_completion_tokens: 2000
```

### Azure OpenAI

```yaml
aiTargets:
  - id: azure-openai-chat
    provider: azure
    apiKey: ${AZURE_OPENAI_API_KEY}
    settings:
      resourceName: ${AZURE_OPENAI_RESOURCE}
      apiVersion: ${AZURE_OPENAI_API_VERSION}
    options:
      model: ${AZURE_OPENAI_DEPLOYMENT}   # Azure addresses models by deployment name
      temperature: 0.7
      max_tokens: 800
```

Microsoft Entra ID instead of a static key:

```yaml
aiTargets:
  - id: azure-openai-entra
    provider: azure
    settings:
      resourceName: ${AZURE_OPENAI_RESOURCE}
      apiVersion: ${AZURE_OPENAI_API_VERSION}
      useEntraId: true
      entraScope: https://cognitiveservices.azure.com/.default   # optional, this is the default
    options:
      model: ${AZURE_OPENAI_DEPLOYMENT}
```

`useEntraId` resolves a token through `DefaultAzureCredential` (managed identity, Azure CLI, or an environment service principal), so no `apiKey` is configured.

### Anthropic

```yaml
aiTargets:
  - id: anthropic
    provider: anthropic
    apiKey: ${ANTHROPIC_API_KEY}
    options:
      model: claude-sonnet-5-5
      max_tokens: 12000
```

### AWS Bedrock

Static keys:

```yaml
aiTargets:
  - id: bedrock
    provider: bedrock
    settings:
      region: ${AWS_BEDROCK_REGION}
      accessKeyId: ${AWS_BEDROCK_ACCESS_KEY_ID}
      secretAccessKey: ${AWS_BEDROCK_SECRET_ACCESS_KEY}
      sessionToken: ${AWS_BEDROCK_SESSION_TOKEN}   # Optional
    options:
      model: "eu.anthropic.claude-haiku-5-5"
      max_tokens: 800
```

Cross-region inference profile (the ARN or profile id goes in `options.model`):

```yaml
aiTargets:
  - id: bedrock-inference-profile
    provider: bedrock
    settings:
      region: eu-north-1
      accessKeyId: ${AWS_BEDROCK_ACCESS_KEY_ID}
      secretAccessKey: ${AWS_BEDROCK_SECRET_ACCESS_KEY}
    requestTimeout: 300000   # 5 minutes for larger models
    options:
      model: "arn:aws:bedrock:eu-north-1:123456789012:inference-profile/eu.anthropic.claude-sonnet-5-5"
      max_tokens: 8000
```

IAM role assumption via STS:

```yaml
aiTargets:
  - id: bedrock-role
    provider: bedrock
    settings:
      region: ${AWS_BEDROCK_REGION}
      roleArn: ${AWS_BEDROCK_ROLE_ARN}
      externalId: ${AWS_BEDROCK_EXTERNAL_ID}   # optional, for cross-account roles
    options:
      model: "eu.anthropic.claude-haiku-5-5"
      max_tokens: 800
```

The STS call itself uses the target's default AWS credential chain; the assumed credentials are cached in memory and refreshed about 60 s before expiry, not re-requested on every call.

Default credential chain (instance/container role, environment variables, shared config or SSO):

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

Credential precedence: static `accessKeyId` + `secretAccessKey` when both are set → otherwise `roleArn` STS assumption → otherwise the default AWS credential chain.

### Google Gemini API

```yaml
aiTargets:
  - id: gemini
    provider: google
    apiKey: ${GEMINI_API_KEY}
    options:
      model: gemini-3.8-flash
```

### Google Vertex AI

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

The default OAuth scope is `https://www.googleapis.com/auth/cloud-platform`; set `settings.googleAuthOptions.scopes: [<scope>]` only to request something else. Literal `\n` sequences in the private key are converted to real newlines automatically.

### Mistral

```yaml
aiTargets:
  - id: mistral
    provider: mistral
    apiKey: ${MISTRAL_API_KEY}
    options:
      model: mistral-small-latest
      max_tokens: 800
```

### OpenRouter

```yaml
aiTargets:
  - id: openrouter
    provider: openrouter
    apiKey: ${OPENROUTER_API_KEY}
    options:
      model: anthropic/claude-sonnet-5.5
```

### Hugging Face

```yaml
aiTargets:
  - id: huggingface-main
    provider: huggingface
    apiKey: ${HUGGINGFACE_API_KEY}
    options:
      model: meta-models/Muse-Glimmer-30B
```

### OpenAI-compatible endpoints

Writer:

```yaml
aiTargets:
  - id: writer
    provider: openai-compatible
    apiKey: ${WRITER_API_KEY}
    settings:
      baseURL: https://api.writer.com/v1
    options:
      model: palmyra-x6
```

Local Ollama:

```yaml
aiTargets:
  - id: local-ollama
    provider: openai-compatible
    settings:
      baseURL: http://localhost:11434/v1
    options:
      model: muse-glimmer
```

Azure AI Inference (OpenAI-v1-compatible endpoints):

```yaml
aiTargets:
  - id: azure-ai-inference-main
    provider: openai-compatible
    apiKey: ${AZURE_AI_INFERENCE_API_KEY}
    settings:
      baseURL: https://<your-resource>.services.ai.azure.com/openai/v1
    options:
      model: <your-deployment-name>
```

## Request Defaults and Precedence

The effective request body is `{ ...target.options, ...requestBody }` — request fields win, including `model`.

Mapped chat-completion parameters:

`model`, `messages`, `tools` (function tools), `tool_choice`, `temperature`, `top_p`, `max_tokens`, `max_completion_tokens` (wins over `max_tokens`), `stop`, `seed`, `frequency_penalty`, `presence_penalty`, `reasoning_effort` (`none` | `minimal` | `low` | `medium` | `high` | `xhigh` | `max`; invalid values return `400` `INVALID_REASONING_EFFORT`), `stream`, `stream_options.include_usage`, `response_format` (`json_object` | `json_schema`), `n`.

Any other body key is dropped, and its name is logged:

```
AI-Engine dropped unsupported params: <names>
```

Provider-specific extras go into the target's `providerOptions`, keyed by provider id and forwarded verbatim to the provider. The example below raises Claude's thinking effort (Claude 5.5 models think adaptively; `effort` steers how much).

```yaml
aiTargets:
  - id: anthropic-thinking
    provider: anthropic
    apiKey: ${ANTHROPIC_API_KEY}
    providerOptions:
      anthropic:
        effort: high
    options:
      model: claude-sonnet-5-5
      max_tokens: 12000
```

## Structured Output

`response_format: json_object` and `response_format: json_schema` are executed — the target model is wrapped so the provider enforces the requested format:

```bash
curl -X POST http://localhost:${PROACTIONS_HUB_PORT}/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "x-target: openai" \
  -d '{
    "messages": [{"role": "user", "content": "Summarize the article in one sentence."}],
    "response_format": {
      "type": "json_schema",
      "json_schema": {
        "name": "summary",
        "schema": {
          "type": "object",
          "properties": {"summary": {"type": "string"}},
          "required": ["summary"],
          "additionalProperties": false
        }
      }
    }
  }'
```

The Responses API supports the equivalent `text.format`.

## Timeouts, Retries and Fallback

Chat completions and the Responses API retry retryable upstream failures by default — 3 total attempts (1 initial + 2 retries) with equal-jitter exponential backoff — configurable globally under `hub.http_client.retry` and per target under `retry`. A route's retries share that route's `requestTimeout` budget, so raise the timeout if you raise the attempt count. Images, speech, transcription and translation make a single attempt.

Full details, including composite `fallback`/`loadbalance` behaviour: [Routing, Retries & Timeouts](./ai-link-routing.md).

## Guardrails

Top-level `guardrails:` entries (regex, length, JSON, PII, LLM judge, moderation API) attach to a concrete `aiTargets` entry by name and run before the request, after the response, or both. A `deny` action returns HTTP `422` with a `guardrail_violation` body.

Full reference: [Guardrails](./ai-link-guardrails.md).

## Errors

| Status | Meaning                                                                                          |
| ------ | ------------------------------------------------------------------------------------------------ |
| `400`  | Validation error — e.g. `MISSING_TARGET`, `INVALID_TARGET`, `MISSING_MODEL`, `MISSING_MESSAGES`, `UNSUPPORTED_MODALITY`, or a legacy-only endpoint on an `aiTargets` id |
| `422`  | Guardrail violation (`guardrail_violation`)                                                       |
| `502`  | Upstream provider failure or timeout                                                              |
| `503`  | Legacy target requested while the legacy AI gateway is not ready                                   |

Upstream failures return a JSON body:

```json
{
  "message": "External service error",
  "target": "openai",
  "provider": "openai",
  "model": "gpt-6-luna",
  "timestamp": "2026-05-04T09:12:31.004Z",
  "endpoint": "https://api.openai.com/v1/chat/completions",
  "statusCode": 500,
  "upstreamError": { "error": { "message": "..." } }
}
```

`endpoint` is the upstream URL. `upstreamError` is the provider's response body (parsed as JSON when possible). `headers` carries `retry-after`, `x-ratelimit-remaining`, `x-ratelimit-reset` and `x-request-id` when the provider sent them.

A timeout returns `message: "Request timeout"` with `error`, `timeoutMs`, `timeoutSeconds` and `suggestion`.

Network failures return `message` `Connection refused`, `DNS resolution failed` or `Connection reset`, with `error`, `code` and `suggestion`.

## Testing Your Configuration

### Test with cURL

```bash
# Test chat completion
curl -X POST http://localhost:${PROACTIONS_HUB_PORT}/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "x-target: openai" \
  -d '{
    "messages": [
      {"role": "user", "content": "Say hello"}
    ]
  }'
```

Add `"stream": true` to the body (and `--no-buffer` to curl) to exercise the streaming path.

### Test with Swagger UI

If Swagger is enabled (`swagger.enabled: true`):

1. Navigate to `http://localhost:${PROACTIONS_HUB_PORT}/docs`
2. Find the `/v1/chat/completions` endpoint
3. Add the `x-target` header
4. Execute the test request

### Verify the target is served by the new engine

An `aiTargets` entry is served in-process, so:

- No `AI-Link request served via deprecated legacy target — migrate '<id>' to aiTargets` warning appears in the logs (log type `aiLink`) for that id.
- Once no legacy `targets:` entries remain, `GET /ready` reports the legacy gateway check (`ai_gateway`) as healthy with `skipped: true`, and no gateway subprocess is started.

## Troubleshooting

### `"<key>" is not allowed` at startup

A legacy provider field was placed at the root of an `aiTargets` entry. Move it under `settings:` (or into `options:` if it is a request parameter) — see the [field mapping table](./ai-link-migration.md#general-field-mapping).

### `INVALID_TARGET`

No `aiTargets` or legacy `targets` entry has the requested id. Check that the `x-target` header (or the ProActions `target:` value) matches an `id` exactly.

### `MISSING_MODEL`

Neither the request body nor the target supplied a model. Set `options.model` on the target, or send `"model"` in the body.

### Provider authentication failed

**Error:** `401 Unauthorized` or `403 Forbidden`.

1. Verify the API key is set and substituted (`${VAR}` resolved, not left literal).
2. Check the provider-specific `settings` (Azure `resourceName`, Bedrock `region`, Vertex `project`/`location`).
3. Ensure the credential has the required permissions (for Bedrock: `bedrock:InvokeModel`; for Vertex: the Vertex AI role on the project).

### Request timeout

**Error:** `502` with `message: "Request timeout"`.

Increase `requestTimeout` on the target:

```yaml
requestTimeout: 300000   # 5 minutes
```

If the target has a blocking post-phase guardrail and serves streams, the timeout covers the whole generation, not just time-to-first-byte — see [Guardrails](./ai-link-guardrails.md).

### Vertex AI: `DECODER routines::unsupported`

The service-account private key is malformed. Literal `\n` sequences are normalized automatically, so check the surrounding quoting: the value must contain the PEM body between `-----BEGIN PRIVATE KEY-----` and `-----END PRIVATE KEY-----` lines with no stray quotes or trailing whitespace.

### Bedrock: `Could not load credentials`

No usable credential was found. Set `settings.accessKeyId` and `settings.secretAccessKey`, or `settings.roleArn`, or give the container a default AWS credential source (instance/container role or environment variables).

## Next Steps

- [Migrating from targets to aiTargets](./ai-link-migration.md)
- [Routing, Retries & Timeouts](./ai-link-routing.md)
- [Guardrails](./ai-link-guardrails.md)
- [Realtime Relay](./realtime.md)
- [Logging](./logging.md)
- [Legacy targets (deprecated)](./ai-link-legacy.md)
