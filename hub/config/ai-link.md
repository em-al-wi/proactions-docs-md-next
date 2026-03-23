---
id: ai-link
title: AI-Link Configuration
sidebar_label: AI-Link
sidebar_position: 3
---

# AI-Link Configuration

The AI-Link module provides a unified gateway to multiple AI service providers through OpenAI-compatible endpoints. This guide covers both quick start and advanced configuration options.

## Quick Start

### Minimal Configuration

The simplest AI-Link configuration requires just an ID, provider, and API key:

```yaml
targets:
  - id: openai
    provider: openai
    apiKey: ${OPENAI_API_KEY}
    options:
      model: gpt-4o-mini
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

The `model` from the target's `options` will be merged with your request.

## Available Endpoints

AI-Link provides OpenAI-compatible endpoints:

- **Chat Completions:** `POST /v1/chat/completions`
- **Image Generation:** `POST /v1/images/generations`
- **Audio Speech:** `POST /v1/audio/speech`
- **Audio Transcription:** `POST /v1/audio/transcriptions`
- **Audio Translation:** `POST /v1/audio/translations`

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
- Client disconnects are detected immediately — the upstream connection is torn down to prevent resource leaks.
- Streaming requests appear separately in the [Admin Panel](../admin-panel.md) AI-Link metrics (streaming ratio).
- Errors that occur mid-stream (after headers are sent) are forwarded as a final `data:` error chunk.

## Target Configuration Structure

Each target in the `targets` array has this structure:

```yaml
targets:
  - id: string              # Required: Unique identifier
    provider: string        # Required (unless using config): Provider name
    apiKey: string         # Required: API key for the provider
    options: object        # Optional: Default request parameters
    requestTimeout: number # Optional: Override default timeout (ms)
    # Provider-specific fields...
```

### Required Fields

| Field      | Description                                                                                         |
| ---------- | --------------------------------------------------------------------------------------------------- |
| `id`       | Unique identifier used in the `x-target` header                                                     |
| `provider` | Provider name (see [Supported Providers](#supported-providers)) OR use `config` for complex routing |
| `apiKey`   | API key for authentication (if required by provider)                                                |

### Optional Fields

| Field            | Description                                 | Default          |
| ---------------- | ------------------------------------------- | ---------------- |
| `options`        | Default parameters merged with each request | `{}`             |
| `requestTimeout` | Request timeout in milliseconds             | `120000` (2 min) |

## Supported Providers

### Major AI Providers

ProActions Hub supports 50+ AI providers. Here are the most commonly used:

| Provider ID     | Name                  | Configuration Options                                                   |
| --------------- | --------------------- | ----------------------------------------------------------------------- |
| `openai`        | OpenAI                | `openaiOrganization`, `openaiProject`                                   |
| `azure-openai`  | Azure OpenAI          | `azureResourceName`, `azureDeploymentId`, `azureApiVersion`             |
| `anthropic`     | Anthropic Claude      | None                                                                    |
| `bedrock`       | AWS Bedrock           | `awsAccessKeyId`, `awsSecretAccessKey`, `awsRegion`, `awsSessionToken`  |
| `vertex-ai`     | Google Vertex AI      | `vertexProjectId`, `vertexRegion`, `privateKey`, `clientEmail`, `scope` |
| `google`        | Google Gemini         | None                                                                    |
| `perplexity-ai` | Perplexity AI         | None                                                                    |
| `openrouter`    | OpenRouter            | None                                                                    |
| `workers-ai`    | Cloudflare Workers AI | `workersAiAccountId`                                                    |
| `mistral-ai`    | Mistral AI            | None                                                                    |
| `groq`          | Groq                  | None                                                                    |
| `fireworks-ai`  | Fireworks AI          | None                                                                    |
| `together-ai`   | Together AI           | None                                                                    |
| `deepseek`      | DeepSeek              | None                                                                    |
| `x-ai`          | xAI (Grok)            | None                                                                    |

<details>
<summary>View all 50+ supported providers</summary>

| Provider ID     | Name             |
| --------------- | ---------------- |
| `ai21`          | AI21             |
| `anyscale`      | Anyscale         |
| `cerebras`      | Cerebras AI      |
| `cohere`        | Cohere           |
| `cortex`        | Cortex AI        |
| `dashscope`     | DashScope        |
| `deepbricks`    | Deepbricks       |
| `deepinfra`     | Deep Infra       |
| `deepseek`      | DeepSeek         |
| `fireworks-ai`  | Fireworks AI     |
| `github`        | GitHub Inference |
| `google`        | Google Gemini    |
| `groq`          | Groq             |
| `inference-net` | Inference.net    |
| `jina`          | Jina             |
| `lambda`        | Lambdalabs       |
| `lemonfox-ai`   | Lemonfox AI      |
| `lingyi`        | Lingyi           |
| `milvus`        | Milvus           |
| `mistral-ai`    | Mistral AI       |
| `monsterapi`    | Monster API      |
| `moonshot`      | Moonshot AI      |
| `nebius`        | Nebius           |
| `nomic`         | Nomic AI         |
| `novita-ai`     | Novita AI        |
| `ollama`        | Ollama           |
| `palm`          | Google PaLM      |
| `predibase`     | Predibase        |
| `recraft-ai`    | Recraft          |
| `reka-ai`       | Reka AI          |
| `replicate`     | Replicate        |
| `sagemaker`     | Amazon SageMaker |
| `sambanova`     | SambaNova        |
| `segmind`       | Segmind          |
| `siliconflow`   | Silicon Flow     |
| `stability-ai`  | Stability AI     |
| `triton`        | NVIDIA Triton    |
| `upstage`       | Upstage AI       |
| `voyage`        | Voyage AI        |
| `zhipu`         | Zhipu            |

</details>

### Custom Providers

For providers not in the list, use the `custom` provider:

```yaml
targets:
  - id: my-custom-provider
    provider: custom
    endpoint: https://api.example.com{path}
    apiKey: ${CUSTOM_API_KEY}
    options:
      model: my-model
```

The `{path}` placeholder will be replaced with the request path (e.g., `/v1/chat/completions`).

## Provider-Specific Configuration

### OpenAI

```yaml
targets:
  - id: openai
    provider: openai
    apiKey: ${OPENAI_API_KEY}
    openaiOrganization: ${OPENAI_ORG}      # Optional
    openaiProject: ${OPENAI_PROJECT}       # Optional
    options:
      model: gpt-4o
      temperature: 0.7
      max_completion_tokens: 2000
```

### Azure OpenAI

```yaml
targets:
  - id: azure-openai
    provider: azure-openai
    apiKey: ${AZURE_OPENAI_API_KEY}
    azureResourceName: ${AZURE_OPENAI_RESOURCE}     # Required
    azureDeploymentId: ${AZURE_OPENAI_DEPLOYMENT}   # Required
    azureApiVersion: ${AZURE_OPENAI_API_VERSION}    # Required (e.g., "2024-02-15-preview")
    options:
      temperature: 0.7
      top_p: 0.95
      max_tokens: 2000
```

Environment variables:

```bash
AZURE_OPENAI_API_KEY=your-key
AZURE_OPENAI_RESOURCE=your-resource-name
AZURE_OPENAI_DEPLOYMENT=gpt-4o
AZURE_OPENAI_API_VERSION=2024-02-15-preview
```

### Anthropic Claude

```yaml
targets:
  - id: anthropic
    provider: anthropic
    apiKey: ${ANTHROPIC_API_KEY}
    options:
      model: claude-4-5-sonnet
      max_tokens: 12000
      temperature: 0.7
```

### AWS Bedrock

```yaml
targets:
  - id: bedrock
    provider: bedrock
    awsAccessKeyId: ${AWS_BEDROCK_ACCESS_KEY_ID}         # Required
    awsSecretAccessKey: ${AWS_BEDROCK_SECRET_ACCESS_KEY} # Required
    awsRegion: ${AWS_BEDROCK_REGION}                     # Required
    awsSessionToken: ${AWS_BEDROCK_SESSION_TOKEN}        # Optional
    options:
      model: anthropic.claude-3-haiku-20240307-v1:0
      max_tokens: 2000
```

For cross-region inference profiles:

```yaml
targets:
  - id: bedrock-inference-profile
    provider: bedrock
    awsAccessKeyId: ${AWS_BEDROCK_ACCESS_KEY_ID}
    awsSecretAccessKey: ${AWS_BEDROCK_SECRET_ACCESS_KEY}
    awsRegion: eu-north-1
    requestTimeout: 300000  # 5 minutes for larger models
    options:
      model: arn:aws:bedrock:eu-north-1:123456789:inference-profile/eu.anthropic.claude-sonnet-4-20250514-v1:0
      max_tokens: 8000
```

### Google Vertex AI

```yaml
targets:
  - id: vertex
    provider: vertex-ai
    vertexProjectId: ${VERTEX_PROJECT_ID}    # Required
    vertexRegion: ${VERTEX_REGION}           # Required
    privateKey: ${VERTEX_PRIVATE_KEY}        # Required: Service account private key
    clientEmail: ${VERTEX_CLIENT_EMAIL}      # Required: Service account email
    scope: ${VERTEX_SCOPE}                   # Required: OAuth scope
    options:
      model: gemini-2.0-flash-exp
```

### Google Gemini

```yaml
targets:
  - id: gemini
    provider: google
    apiKey: ${GEMINI_API_KEY}
    options:
      model: gemini-2.0-flash-exp
```

### Perplexity AI

```yaml
targets:
  - id: perplexity
    provider: perplexity-ai
    apiKey: ${PERPLEXITY_API_KEY}
    options:
      model: sonar
      temperature: 0.7
      stream: false
```

### OpenRouter

```yaml
targets:
  - id: openrouter
    provider: openrouter
    apiKey: ${OPENROUTER_API_KEY}
    options:
      model: anthropic/claude-3.5-sonnet
```

### Cloudflare Workers AI

```yaml
targets:
  - id: workers-ai
    provider: workers-ai
    apiKey: ${WORKERS_AI_API_KEY}
    workersAiAccountId: ${WORKERS_AI_ACCOUNT_ID}  # Required
    options:
      model: @cf/meta/llama-3-8b-instruct
```

### Custom Provider with Endpoint

For any custom OpenAI-compatible API:

```yaml
targets:
  - id: custom-api
    provider: custom
    endpoint: https://api.example.com{path}
    apiKey: ${CUSTOM_API_KEY}
    headers:                              # Optional custom headers
      X-Custom-Header: value
    options:
      model: my-model
```

The `{path}` placeholder will be replaced with the actual endpoint path.

## Advanced Configuration

### Request Timeout Override

Override the default 2-minute timeout for specific targets:

```yaml
targets:
  - id: slow-model
    provider: openai
    apiKey: ${OPENAI_API_KEY}
    requestTimeout: 300000  # 5 minutes
    options:
      model: o1-preview
```

### Default Options

Options specified in the target configuration are merged with request parameters:

```yaml
targets:
  - id: openai-conservative
    provider: openai
    apiKey: ${OPENAI_API_KEY}
    options:
      model: gpt-4o
      temperature: 0.1
      max_completion_tokens: 500
      top_p: 0.9
```

Request parameters override defaults:

```json
{
  "messages": [...],
  "temperature": 0.8  // This overrides the default 0.1
}
```

### Multiple Targets for Same Provider

You can configure multiple targets for the same provider with different settings:

```yaml
targets:
  - id: openai-fast
    provider: openai
    apiKey: ${OPENAI_API_KEY}
    options:
      model: gpt-4o-mini
      max_completion_tokens: 500

  - id: openai-quality
    provider: openai
    apiKey: ${OPENAI_API_KEY}
    options:
      model: gpt-4o
      max_completion_tokens: 4000
      temperature: 0.3
```

### Load Balancing Configuration

For advanced routing strategies, use the `config` field with AI-Link's native configuration:

```yaml
targets:
  - id: load-balanced
    config:
      strategy:
        mode: loadbalance
      targets:
        - provider: openai
          api_key: ${OPENAI_API_KEY}
          override_params:
            model: gpt-4o-mini
        - provider: anthropic
          api_key: ${ANTHROPIC_API_KEY}
          override_params:
            model: claude-3-5-haiku-20241022
```

:::info Advanced Configuration
When using `config`, you cannot use the `provider` field. The `config` object is passed directly to AI-Link for advanced routing strategies.
:::

### Failover Configuration

Configure automatic failover between providers:

```yaml
targets:
  - id: with-failover
    config:
      strategy:
        mode: fallback
      targets:
        - provider: openai
          api_key: ${OPENAI_API_KEY}
          override_params:
            model: gpt-4o
        - provider: anthropic
          api_key: ${ANTHROPIC_API_KEY}
          override_params:
            model: claude-3-5-sonnet-20241022
```

### Guardrails

Add content moderation hooks:

```yaml
targets:
  - id: with-guardrails
    config:
      provider: openai
      api_key: ${OPENAI_API_KEY}
      beforeRequestHooks:
        - type: guardrail
          id: content_check
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

## Configuration Best Practices

### Security

1. **Never hardcode API keys** - Always use environment variables
2. **Use environment-specific configs** - Separate dev/staging/production
3. **Limit default max_tokens** - Prevent unexpected costs
4. **Set reasonable timeouts** - Avoid hanging requests

### Performance

1. **Configure appropriate timeouts** - Longer for complex models
2. **Use specific model names** - Don't rely on provider defaults
3. **Set conservative defaults** - Override in requests when needed

### Cost Control

1. **Set max_tokens limits** - Control maximum cost per request
2. **Use cheaper models by default** - Override for quality when needed
3. **Monitor token usage** - Enable AI-Link logging (see [Logging Configuration](logging.md))

### Reliability

1. **Configure multiple targets** - Enable failover scenarios
2. **Set appropriate timeouts** - Balance responsiveness and success rate
3. **Use load balancing** - Distribute load across providers

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

### Test with Swagger UI

If Swagger is enabled (`swagger.enabled: true`):

1. Navigate to `http://localhost:${PROACTIONS_HUB_PORT}/docs`
2. Find the `/v1/chat/completions` endpoint
3. Add `x-target` header
4. Execute test request

### Verify Token Logging

Enable AI-Link logging to verify token usage tracking:

```yaml
logging:
  types:
    aiLink:
      enabled: true
      console: true
      logPrompt: false
      logResponse: false
```

Check logs for token usage:

```json
{
  "level": "info",
  "message": "AI request completed",
  "target": "openai",
  "model": "gpt-4o-mini",
  "promptTokens": 15,
  "completionTokens": 25,
  "totalTokens": 40
}
```

## Troubleshooting

### Target Not Found

**Error:** `Target not found: xyz`

**Solution:** Verify the target ID in config.yaml matches the `x-target` header value.

### Provider Authentication Failed

**Error:** `401 Unauthorized` or `403 Forbidden`

**Solution:**

1. Verify API key is set correctly
2. Check provider-specific fields (deployment ID, region, etc.)
3. Ensure API key has necessary permissions

### Request Timeout

**Error:** `Request timeout after 120000ms`

**Solution:** Increase `requestTimeout` for the target:

```yaml
requestTimeout: 300000  # 5 minutes
```

### Invalid Model

**Error:** `Model not found` or `Invalid model`

**Solution:** Verify the model name is correct for the provider. Check provider documentation.

### Azure OpenAI Configuration

**Error:** `Missing required field: azureDeploymentId`

**Solution:** Azure OpenAI requires three additional fields:

```yaml
azureResourceName: your-resource
azureDeploymentId: your-deployment
azureApiVersion: 2024-02-15-preview
```

## Next Steps

- [Configure Proxy targets](proxy.md) for HTTP forwarding
- [Set up YouTube integration](youtube.md) for video uploads
- [Configure Logging](logging.md) for token usage monitoring
