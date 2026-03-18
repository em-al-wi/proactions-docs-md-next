# Service Configuration

This guide explains how to configure external services in ProActions, including AI providers, translation services, and other APIs.

## Service Configuration Overview

Services in ProActions define how workflows connect to external APIs. Configuration includes:

- **Endpoints**: API URLs
- **Authentication**: API keys and tokens
- **Default Settings**: Shared configuration across actions

## Service Configuration File

**File:** `/SysConfig/ProActions/services.ai-kit.services.yaml`

This file is included in the main configuration:

```yaml
# /SysConfig/pro-actions.ai-kit.yaml
AI_KIT:
  SERVICES:
    !include /SysConfig/ProActions/services.ai-kit.services.yaml

  SECTIONS:
    # sections...
```

## ProActions Hub (Recommended)

ProActions Hub is the recommended way to configure services. It provides centralized API key management and support for 40+ AI providers.

### Basic Hub Configuration

```yaml
HUB:
  endpoint: "{BASE_URL}/swing/proactions"
  target: openai
  defaultBehavior: |
    You are an assistant who helps journalists write their content.
    Answer concisely and professionally.
    Answer in English unless instructed otherwise.
```

### Hub Configuration Properties

| Property          | Description                             | Required | Example                            |
| ----------------- | --------------------------------------- | -------- | ---------------------------------- |
| `endpoint`        | ProActions Hub URL                      | Yes      | `"{BASE_URL}/swing/proactions"`    |
| `target`          | AI service name configured in Hub       | Yes      | `"openai"`, `"claude"`, `"gemini"` |
| `defaultBehavior` | Default system prompt for all LLM calls | No       | See examples below                 |

### Multiple Hub Targets

Configure different Hub targets for different use cases:

```yaml
HUB:
  endpoint: "{BASE_URL}/swing/proactions"
  target: openai
  defaultBehavior: |
    You are a professional writing assistant.

HUB_CLAUDE:
  endpoint: "{BASE_URL}/swing/proactions"
  target: claude
  defaultBehavior: |
    You are Claude, an AI assistant focused on accuracy.
```

### Supported Hub Targets

ProActions Hub supports many providers, but the available `targets` are managed in the ProActions Hub configuration. Please contact your ProActions Hub administrator to find out which targets have been set up.

## DeepL Translation

DeepL provides high-quality translation services.

### DeepL Configuration

```yaml
DEEPL:
  endpoint: /swing/proactions/proxy/forward/deepl
```

### DeepL Write Configuration

For text improvement and rephrasing:

```yaml
DEEPL_WRITE:
  endpoint: /swing/proactions/proxy/forward/deepl-write
```

### DeepL with Direct API Key (Not Recommended)

```yaml
DEEPL:
  apiKey: "your-deepl-api-key"  # Better: use ProActions Hub
  endpoint: "https://api.deepl.com/v2"
```

**Warning:** Storing API keys in configuration files exposes them in client communications. Use ProActions Hub instead.

## ElevenLabs Voice Services

ElevenLabs provides text-to-speech and speech-to-text services.

### ElevenLabs Configuration

```yaml
ELEVENLABS:
  apiKey: "your-api-key"  # Better: configure via ProActions Hub
  endpoint: "https://api.elevenlabs.io/v1"
```

### Using ElevenLabs through Hub

Configure as a proxy in ProActions Hub for better security:

```yaml
# ProActions Hub config.yaml
proxies:
  - id: elevenlabs
    url: https://api.elevenlabs.io/v1
    options:
      secure: true
      changeOrigin: true
      headers:
        xi-api-key: ${ELEVENLABS_KEY}
```

Then in ProActions:

```yaml
ELEVENLABS:
  endpoint: /swing/proactions/proxy/forward/elevenlabs
```

## Stability AI

Stability AI provides image generation and manipulation services.

### Stability AI Configuration

```yaml
STABILITY_AI:
  apiKey: "your-api-key"  # Better: configure via ProActions Hub
  endpoint: "https://api.stability.ai"
```

### Using Stability AI through Hub

```yaml
# ProActions Hub config.yaml
proxies:
  - id: stability-ai
    url: https://api.stability.ai
    options:
      secure: true
      changeOrigin: true
      headers:
        Authorization: Bearer ${STABILITY_AI_KEY}
```

```yaml
# ProActions configuration
STABILITY_AI:
  endpoint: /swing/proactions/proxy/forward/stability-ai
```

## Brighter AI

Brighter AI provides image anonymization (face and license plate blurring).

### Brighter AI Configuration

```yaml
BRIGHTER_AI:
  apiKey: "your-api-key"
  endpoint: "https://api.brighter.ai"
```

## Direct OpenAI Integration (Not Recommended)

While possible, direct OpenAI integration is not recommended. Use ProActions Hub instead.

### Direct OpenAI Configuration

```yaml
OPENAI:
  apiKey: "sk-..."  # Exposed API key - not secure
  endpoint: "https://api.openai.com/v1"
```

### Why Use Hub Instead?

1. **Security**: API keys stored on server, not in configuration files
2. **Flexibility**: Switch providers without changing workflows
3. **Control**: Centralized access control and monitoring
4. **Proxying**: Handle rate limits and failover

## Service-Specific Settings

### Default LLM Behavior

Configure default system prompts for consistent AI responses:

```yaml
HUB:
  endpoint: "{BASE_URL}/swing/proactions"
  target: openai
  defaultBehavior: |
    # Role
    You are an expert assistant for professional journalists.

    # Guidelines
    - Provide accurate, well-researched information
    - Write in a clear, professional style
    - Avoid speculation and clearly mark opinions
    - Cite sources when possible

    # Output
    - Answer concisely unless detail is requested
    - Use proper grammar and punctuation
    - Default to English unless another language is specified
```

### Language-Specific Behaviors

```yaml
HUB_FRENCH:
  endpoint: "{BASE_URL}/swing/proactions"
  target: openai
  defaultBehavior: |
    Vous êtes un assistant pour les journalistes francophones.
    Répondez toujours en français avec un style professionnel.
```

### Domain-Specific Behaviors

```yaml
HUB_SPORTS:
  endpoint: "{BASE_URL}/swing/proactions"
  target: openai
  defaultBehavior: |
    You are a sports journalism assistant.
    Focus on accuracy, statistics, and engaging storytelling.
    Use appropriate sports terminology.

HUB_FINANCE:
  endpoint: "{BASE_URL}/swing/proactions"
  target: openai
  defaultBehavior: |
    You are a financial journalism assistant.
    Prioritize accuracy and clarity when discussing financial topics.
    Explain complex concepts in accessible terms.
```

## Complete Service Configuration Examples

### Minimal Configuration

```yaml
# /SysConfig/ProActions/services.ai-kit.services.yaml
HUB:
  endpoint: "{BASE_URL}/swing/proactions"
  target: openai

DEEPL:
  endpoint: /swing/proactions/proxy/forward/deepl
```

### Standard Configuration

```yaml
# /SysConfig/ProActions/services.ai-kit.services.yaml
HUB:
  endpoint: "{BASE_URL}/swing/proactions"
  target: openai
  defaultBehavior: |
    You are an assistant for professional journalists.
    Provide accurate, concise responses.
    Default to English unless specified otherwise.

DEEPL:
  endpoint: /swing/proactions/proxy/forward/deepl

DEEPL_WRITE:
  endpoint: /swing/proactions/proxy/forward/deepl-write

ELEVENLABS:
  endpoint: /swing/proactions/proxy/forward/elevenlabs
```

### Advanced Multi-Service Configuration

```yaml
# /SysConfig/ProActions/services.ai-kit.services.yaml

# Primary LLM service
HUB:
  endpoint: "{BASE_URL}/swing/proactions"
  target: openai
  defaultBehavior: |
    You are a professional writing assistant for journalists.

# Alternative LLM for specific tasks
HUB_CLAUDE:
  endpoint: "{BASE_URL}/swing/proactions"
  target: claude
  defaultBehavior: |
    You are Claude, focused on thoughtful, accurate responses.

# Translation services
DEEPL:
  endpoint: /swing/proactions/proxy/forward/deepl

DEEPL_WRITE:
  endpoint: /swing/proactions/proxy/forward/deepl-write

# Voice services
ELEVENLABS:
  endpoint: /swing/proactions/proxy/forward/elevenlabs

# Image services
STABILITY_AI:
  endpoint: /swing/proactions/proxy/forward/stability-ai

BRIGHTER_AI:
  endpoint: /swing/proactions/proxy/forward/brighter-ai
```

## Using Services in Workflows

### Using Hub Services

```yaml
- title: 'Generate Headline'
  flow:
    - step: HUB_COMPLETION # will use HUB service
      instruction: "Generate headline for: {textContent}"
```

### Using Alternative Hub Target

```yaml
- title: 'Analyze with Claude'
  flow:
    - step: HUB_COMPLETION
      service: HUB_CLAUDE  # Use alternative service
      instruction: "Analyze: {textContent}"
```

### Using DeepL

```yaml
- title: 'Translate to French'
  flow:
    - step: DEEPL_TRANSLATE
      instruction: "{selectedText}"
      target_lang: "FR"
```

### Using ElevenLabs

```yaml
- title: 'Generate Speech'
  flow:
    - step: SET
      text: "{selectedText}"
    - step: ELEVENLABS_TTS
      voice_id: "21m00Tcm4TlvDq8ikWAM"
```

## Service Configuration Best Practices

### 1. Use ProActions Hub

**Do this:**

```yaml
HUB:
  endpoint: "{BASE_URL}/swing/proactions"
  target: openai
```

**Avoid this:**

```yaml
OPENAI:
  apiKey: "sk-..."  # Exposed in configuration
```

### 2. Organize by Purpose

```yaml
# Clear service organization
HUB:              # General AI
HUB_TRANSLATION:  # Translation-specific AI
HUB_IMAGES:       # Image generation AI
DEEPL:            # Translation service
ELEVENLABS:       # Voice service
```

### 3. Use Descriptive Default Behaviors

```yaml
HUB:
  defaultBehavior: |
    # Clear role definition
    You are an assistant for professional journalists.

    # Specific guidelines
    - Provide factual, well-researched information
    - Use clear, professional language
    - Cite sources when appropriate

    # Output format
    - Be concise unless detail is requested
    - Default to English
```

### 4. Document Service Configuration

```yaml
# Translation service via ProActions Hub
# Configured in Hub with DeepL API key
DEEPL:
  endpoint: /swing/proactions/proxy/forward/deepl

# Alternative: Direct DeepL (requires API key in config)
# DEEPL:
#   apiKey: "..."
#   endpoint: "https://api.deepl.com/v2"
```

### 5. Use Separate Service Files

```yaml
# /SysConfig/pro-actions.ai-kit.yaml
AI_KIT:
  SERVICES:
    !include /SysConfig/ProActions/services.ai-kit.services.yaml
```

This keeps the main configuration clean and services easy to update.

## Troubleshooting Services

### Service Not Responding

**Check:**

- ProActions Hub is running
- Endpoint URLs are correct
- API keys are valid (in Hub configuration)
- Network connectivity

**Debug:**

```yaml
- step: TRY
  try:
    - step: HUB_COMPLETION
      instruction: "Test"
  catch:
    - step: SHOW_NOTIFICATION
      message: "Service error: Check Hub configuration"
```

### API Key Errors

**Symptoms:**

- 401 Unauthorized errors
- 403 Forbidden errors

**Solutions:**

- Verify API key in ProActions Hub configuration
- Check API key has required permissions
- Ensure API key is not expired
- Verify environment variables are loaded

### Rate Limiting

**Symptoms:**

- 429 Too Many Requests errors
- Slow responses

**Solutions:**

- Implement delays with SLEEP step
- Use TRY/catch for retries
- Contact Hub administrator for rate limit adjustments

### Proxy Issues

**Symptoms:**

- Cannot reach service through Hub
- Timeout errors

**Solutions:**

- Verify proxy configuration in Hub
- Check firewall settings
- Test direct service access
- Review Hub logs

## Testing Service Configuration

### Test Basic Connectivity

```yaml
- title: 'Test Hub Connection'
  flow:
    - step: HUB_COMPLETION
      instruction: "Say hello"
      max_tokens: 10

    - step: SHOW_NOTIFICATION
      message: "Service working: {{ flowContext.text }}"
```

### Test Each Service

```yaml
- title: 'Test Services'
  flow:
    # Test LLM
    - step: HUB_COMPLETION
      instruction: "Test"

    # Test Translation
    - step: DEEPL_TRANSLATE
      instruction: "Hello"
      target_lang: "FR"

    # Show results
    - step: SHOW_NOTIFICATION
      message: "All services operational"
```

## Next Steps

- **[Configuration Basics](./basics.md)** - Overall configuration concepts
- **[Forms and Inputs](./forms-and-inputs.md)** - Create interactive forms
- **[Menus and Triggers](./menus-and-triggers.md)** - Configure menus and shortcuts
- **[Using Services Guide](../../getting-started/using-services.md)** - Using services in workflows
