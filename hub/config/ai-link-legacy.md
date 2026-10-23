---
id: ai-link-legacy
title: Legacy AI-Link Targets (Deprecated)
sidebar_label: Legacy targets (deprecated)
---

# Legacy AI-Link Targets (Deprecated)

:::warning[Deprecated]
`targets:` entries are served by the legacy AI gateway, a separate subprocess, and are deprecated since Hub 1.3.0. They keep working in 1.3.x, but every request logs a deprecation warning, and support will be removed in a future release.

Migrate to [`aiTargets`](./ai-link.md) using the [migration guide](./ai-link-migration.md).
:::

This page documents the legacy `targets:` section exactly as it behaves today, for operators who have not migrated yet. New deployments should configure [`aiTargets`](./ai-link.md) instead.

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

The legacy AI gateway accepts a catalog of 50+ providers. Here are the most commonly used:

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
      model: gpt-6.1-sol
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
AZURE_OPENAI_DEPLOYMENT=gpt-6.1-sol
AZURE_OPENAI_API_VERSION=2024-02-15-preview
```

### Anthropic Claude

```yaml
targets:
  - id: anthropic
    provider: anthropic
    apiKey: ${ANTHROPIC_API_KEY}
    options:
      model: claude-sonnet-5-5
      max_tokens: 12000
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
      model: anthropic.claude-haiku-5-5
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
      model: arn:aws:bedrock:eu-north-1:123456789012:inference-profile/eu.anthropic.claude-sonnet-5-5
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
      model: gemini-3.8-flash
```

### Google Gemini

```yaml
targets:
  - id: gemini
    provider: google
    apiKey: ${GEMINI_API_KEY}
    options:
      model: gemini-3.8-flash
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
      model: anthropic/claude-sonnet-5.5
```

### Cloudflare Workers AI

```yaml
targets:
  - id: workers-ai
    provider: workers-ai
    apiKey: ${WORKERS_AI_API_KEY}
    workersAiAccountId: ${WORKERS_AI_ACCOUNT_ID}  # Required
    options:
      model: '@cf/meta/llama-4-scout-17b-16e-instruct'
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
      model: gpt-6-astra
```

### Default Options

Options specified in the target configuration are merged with request parameters:

```yaml
targets:
  - id: openai-conservative
    provider: openai
    apiKey: ${OPENAI_API_KEY}
    options:
      model: gpt-6-luna
      reasoning_effort: none   # GPT-6 models accept temperature/top_p only without reasoning
      temperature: 0.1
      max_completion_tokens: 500
      top_p: 0.9
```

Request parameters override defaults:

```json
{
  "messages": [...],
  "temperature": 0.8
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
      model: gpt-6-luna
      max_completion_tokens: 500

  - id: openai-quality
    provider: openai
    apiKey: ${OPENAI_API_KEY}
    options:
      model: gpt-6.1-sol
      max_completion_tokens: 4000
```

### Load Balancing Configuration

For advanced routing strategies, use the `config` field with a legacy routing object:

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
            model: gpt-6-luna
        - provider: anthropic
          api_key: ${ANTHROPIC_API_KEY}
          override_params:
            model: claude-haiku-5-5
```

:::info[Advanced Configuration]
When using `config`, you cannot use the `provider` field. The `config` object is passed directly to the legacy AI gateway for advanced routing strategies.
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
            model: gpt-6.1-sol
        - provider: anthropic
          api_key: ${ANTHROPIC_API_KEY}
          override_params:
            model: claude-sonnet-5-5
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

## Legacy Gateway Settings

The following `hub` settings configure the legacy AI gateway subprocess. They are **only** used while at least one legacy target exists — an `aiTargets`-only deployment never spawns the subprocess, and these settings have no effect.

| Setting                         | Default     | Description                                                  |
| ------------------------------- | ----------- | ------------------------------------------------------------ |
| `hub.ailink_port`               | `3001`      | Port the legacy gateway listens on                                   |
| `hub.ailink_host`               | `localhost` | Host the legacy gateway binds to                                     |
| `hub.ailink_protocol`           | `http`      | Protocol used to reach the legacy gateway                            |
| `hub.ailink_health_check_interval` | `30000`  | Health probe interval (ms)                                    |
| `hub.ailink_max_restart_attempts`  | `3`      | Restart attempts after a crash                                |
| `hub.ailink_max_consecutive_failures` | `3`   | Consecutive health failures before a restart                  |
| `hub.ailink_restart_delay`      | `5000`      | Delay between restart attempts (ms)                           |
| `hub.ailink_startup_timeout`    | `30000`     | How long to wait for the legacy gateway to become ready (ms)         |

Each of these can be overridden through the matching `PROACTIONS_HUB_AILINK_*` environment variable.

:::tip[Removing the legacy gateway]
Once every legacy target is migrated, delete the `targets:` section (or leave it empty). On the next restart the Hub does not start the legacy AI gateway at all, and `/ready` reports its `ai_gateway` check as `skipped: true`.
:::

## Next Steps

- [Migrate to `aiTargets`](./ai-link-migration.md)
- [AI-Link configuration for `aiTargets`](./ai-link.md)
