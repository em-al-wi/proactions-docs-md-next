# Using Services

ProActions integrates with numerous external services, from AI providers to translation services and image processing tools. This guide explains how to configure and use these services in your workflows.

## ProActions Hub: The Recommended Approach

**ProActions Hub** is the recommended way to integrate with external services. It provides:

- **Centralized API key management** - Keep API keys secure and separate from configuration
- **Unified interface** - Use the same configuration across different AI providers
- **EDAPI authentication** - Protect API usage behind your authentication system
- **Proxy support** - Route requests through specified proxies
- **Content extraction** - Built-in tool for extracting data from URLs

### Why Use ProActions Hub?

Instead of embedding API keys directly in your YAML configuration files, ProActions Hub:

1. Stores API keys securely on the server
2. Allows administrators to manage keys centrally
3. Prevents accidental exposure of credentials in configuration files
4. Provides access control through EDAPI authentication

## Service Configuration

### Setting Up ProActions Hub

In your `/SysConfig/pro-actions.ai-kit.yaml`:

```yaml
AI_KIT:
  SERVICES:
    HUB:
      endpoint: "{BASE_URL}/swing/proactions"
      target: openai
      defaultBehavior: |
        You are an assistant who helps journalists write their content.
        Answer without unnecessary explanations.
        Answer in English unless instructed otherwise.
```

#### Configuration Options

- **endpoint**: The ProActions Hub endpoint URL (use `{BASE_URL}` for the current instance)
- **target**: The AI service to use (configured in ProActions Hub)
- **defaultBehavior**: Default system prompt applied to all LLM calls

### Supported Services via Hub

ProActions Hub normalizes access to over 40+ AI providers, including:

**Major AI Providers:**

- OpenAI
- Azure OpenAI
- Anthropic (Claude)
- Google Gemini
- AWS Bedrock
- Mistral AI
- Cohere
- xAI

**Image Processing:**

- Stability AI
- Brighter AI (anonymization)

**And many more...**

Check with your ProActions Hub administrator for available configured targets.

## Using LLM Services

### HUB_COMPLETION: Chat and Text Generation

The most commonly used step for AI interactions:

```yaml
- step: HUB_COMPLETION
  behavior: |
    You are an expert journalist assistant.
    Generate engaging content based on the user's request.
  instruction: "Write a headline for this article: {textContent}"
```

#### Configuration Options

```yaml
- step: HUB_COMPLETION
  # Prompts
  behavior: "System prompt defining AI behavior"
  instruction: "User prompt with the task"

  options:
    # Model settings
    model: "gpt-4"  # Optional, defaults to Hub configuration
    max_tokens: 500
    temperature: 0.7  # 0.0 = deterministic, 1.0 = creative

    # Advanced
    seed: 42  # For reproducible outputs
    top_p: 0.9
    frequency_penalty: 0.0
    presence_penalty: 0.0
```

### Structured Outputs

ProActions supports structured AI responses:

#### List Format

```yaml
- step: HUB_COMPLETION
  instruction: "Generate 5 article topics about {textContent}"
  response_format: "list"

- step: USER_SELECT
  promptText: "Choose a topic:"
```

#### JSON Object

```yaml
- step: HUB_COMPLETION
  instruction: |
    Analyze this article and return:
    - sentiment (positive/negative/neutral)
    - main_topics (array of strings)
    - word_count_estimate (number)
  response_format: "json_object"

- step: SHOW_NOTIFICATION
  message: "Sentiment: {{ flowContext.sentiment }}"
```

### HUB_TRANSCRIPTION: Speech-to-Text

Convert audio to text:

```yaml
- step: FILE_UPLOAD
  accept: "audio/*"

- step: HUB_TRANSCRIPTION
  language: "en"  # Optional

- step: INSERT_TEXT
  at: CURSOR
```

### HUB_SPEECH: Text-to-Speech

Generate audio from text:

```yaml
- step: HUB_SPEECH
  voice: "alloy"  # Options: alloy, echo, fable, onyx, nova, shimmer
  speed: 1.0

- step: PLAY_AUDIO
```

### HUB_IMAGE_GENERATION: Generate Images

Create images with GPT-Image-1 or DALL·E:

```yaml
- step: PROMPT
  promptText: "Describe the image you want to generate:"

- step: HUB_IMAGE_GENERATION
  prompt: "{{ flowContext.text }}"
  model: "dall-e-3"
  size: "1024x1024"
  quality: "standard"  # or "hd"

- step: IMAGE_PICKER
```

### HUB_CONTENT_EXTRACTION: Extract Web Content

Extract text content from URLs:

```yaml
- step: PROMPT
  promptText: "Enter URL to extract content from:"

- step: HUB_CONTENT_EXTRACTION
  url: "{{ flowContext.text }}"

- step: HUB_COMPLETION
  instruction: "Summarize this content: {{ flowContext.object.content }}"
```

## Translation Services

### DeepL Translation

ProActions integrates with DeepL for high-quality translation:

```yaml
AI_KIT:
  SERVICES:
    DEEPL:
      endpoint: /swing/proactions/proxy/forward/deepl
```

#### DEEPL_TRANSLATE: Translate Text

```yaml
- step: DEEPL_TRANSLATE
  instruction: "{selectedText}"
  target_lang: "FR"  # French
  source_lang: "EN"  # Optional, auto-detect if omitted
  formality: "default"  # Options: default, more, less, prefer_more, prefer_less

- step: REPLACE_TEXT
  at: CURSOR
```

#### Supported Languages

Common target languages:

- `EN` (English), `FR` (French), `DE` (German), `ES` (Spanish)
- `IT` (Italian), `PT` (Portuguese), `NL` (Dutch), `PL` (Polish)
- `RU` (Russian), `JA` (Japanese), `ZH` (Chinese)
- And many more...

#### DEEPL_WRITE: Rephrase Text

Improve and rephrase text:

```yaml
- step: SET
  text: "{selectedText}"

- step: DEEPL_WRITE

- step: REPLACE_TEXT
  at: CURSOR
```

## Voice Services

### ElevenLabs Text-to-Speech

High-quality voice synthesis:

```yaml
AI_KIT:
  SERVICES:
    ELEVENLABS:
      # Better: configure API key via ProActions Hub
      apiKey: "your-api-key"
      endpoint: "https://api.elevenlabs.io/v1"
```

```yaml
- step: SET
  text: "{selectedText}"

- step: ELEVENLABS_TTS
  voice_id: "21m00Tcm4TlvDq8ikWAM"  # Rachel voice
  model_id: "eleven_monolingual_v1"
  stability: 0.5
  similarity_boost: 0.75

- step: DOWNLOAD
  filename: "speech.mp3"
```

### ElevenLabs Speech-to-Text

```yaml
- step: FILE_UPLOAD
  accept: "audio/*"

- step: ELEVENLABS_STT

- step: INSERT_TEXT
  at: CURSOR
```

## Image Processing

### Stability AI

#### Upscale Images

```yaml
- step: SELECTED_OBJECT

- step: EDAPI_OBJECT_CONTENT

- step: STABILITY_AI_UPSCALE

- step: UPDATE_BINARY_CONTENT
```

#### Search and Replace in Images

```yaml
- step: STABILITY_AI_SEARCH_AND_REPLACE
  prompt: "a red car"
  search_prompt: "blue car"
```

#### Outpaint Images

```yaml
- step: STABILITY_AI_OUTPAINT
  left: 100
  right: 100
  up: 50
  down: 50
  prompt: "extend the beach scene"
```

### Brighter AI: Image Anonymization

Automatically blur faces and license plates:

```yaml
- step: SELECTED_OBJECT

- step: EDAPI_OBJECT_CONTENT

- step: BRIGHTER_AI

- step: UPDATE_BINARY_CONTENT
```

## Generic REST API Integration

For services not directly supported, use the REST step:

```yaml
- step: REST
  url: "https://api.example.com/endpoint"
  method: "POST"
  headers:
    Content-Type: "application/json"
    Authorization: "Bearer {{ flowContext.apiToken }}"
  body:
    query: "{selectedText}"
    language: "en"
```

### REST Configuration Options

```yaml
- step: REST
  url: "https://api.example.com/data"
  method: "GET"  # GET, POST, PUT, DELETE, PATCH

  # Headers
  headers:
    Authorization: "Bearer {{ flowContext.token }}"
    Custom-Header: "value"

  # Query parameters
  parameters:
    param1: "value1"
    param2: "{{ flowContext.variable }}"

  # Request body (for POST/PUT/PATCH)
  body:
    key: "value"
    data: "{{ flowContext.myData }}"

  # Or form data
  formData:
    field1: "value1"
```

## Service Configuration Best Practices

### 1. Use ProActions Hub for API Keys

**Don't do this:**

```yaml
SERVICES:
  OPENAI:
    apiKey: "sk-proj-xxxxxxxxxxxxxxxx"  # Exposed in config and client communication!
```

**Do this:**

```yaml
SERVICES:
  HUB:
    endpoint: "{BASE_URL}/swing/proactions"
    target: openai  # API key managed in Hub
```

### 2. Set Reasonable Limits

```yaml
- step: HUB_COMPLETION
  instruction: "{textContent}"
  options:
    max_tokens: 8000  # Prevent excessive API usage
    temperature: 0.7  # Balance creativity and consistency
```

### 3. Handle Service Errors

```yaml
- step: TRY
  try:
    - step: HUB_COMPLETION
      instruction: "Process: {textContent}"
  catch:
    - step: SHOW_NOTIFICATION
      message: "Service temporarily unavailable. Please try again."
```

### 4. Show Progress for Long Operations

```yaml
- step: SHOW_PROGRESS
  status: '(1/3) Waiting for AI process...'

# do something

- step: UPDATE_PROGRESS
  percentage: '30'
  status: '(2/3) Waiting for AI process...'

# do something else

- step: UPDATE_PROGRESS
  percentage: '90'
  status: '(3/3) Waiting for final step...'

# do something else

- step: HIDE_PROGRESS
```

## Working with Service Outputs

Different steps store their results in different ways:

### Automatic Output Flow

Many steps automatically pass their output to the next step:

```yaml
# HUB_COMPLETION output automatically flows to REPLACE_TEXT
- step: HUB_COMPLETION
  instruction: "Improve this: {selectedText}"

- step: REPLACE_TEXT
  at: CURSOR
```

### Named Outputs

Some steps allow custom output names via the `outputs` configuration:

```yaml
- step: SELECTED_OBJECT
  outputs:
    - type: object
      name: selectedObject

- step: SHOW_NOTIFICATION
  message: "Selected: {{ flowContext.selectedObject.name }}"
```

### Using Client Methods

Access document data using template syntax:

```yaml
- step: SET
  text: '{{ client.getTextContent({ excludeTags: ["caption"] }) | safe }}'
  filename: "{{ client.getMetadata().metadata.general.title }}.mp3"

- step: HUB_SPEECH

- step: UPLOAD
  options:
    name: "{{ flowContext.filename }}"
    workFolder: "{{ client.getDocumentWorkfolder() }}"
```

## Next Steps

- Review the complete [Step Library](../step-library/overview.md) for all service steps
- Learn about [Service Configuration](../guides/configuration/service-configuration.md)
- Explore [Examples](../guides/examples/headline-suggestions.md) for real-world workflows
- Check [Best Practices](../guides/configuration/workflow-design.md) for optimal service usage
