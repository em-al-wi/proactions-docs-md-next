# AI Integration Guide

This guide covers the comprehensive AI integration capabilities provided by the `OPENAI_COMPLETION`, `HUB_COMPLETION`, and `AZURE_OPENAI_COMPLETION` steps. These steps provide a powerful interface to Large Language Models (LLMs) with support for text, images, audio, tool calling, and structured outputs.

## Overview

ProActions provides three completion step types that share the same underlying implementation:

- **`HUB_COMPLETION`** - Routes through ProActions Hub service
- **`OPENAI_COMPLETION`** - Direct OpenAI API integration
- **`AZURE_OPENAI_COMPLETION`** - Azure OpenAI service integration

All three steps support the same features and configuration options, differing only in their default service names.

## Basic Usage

### Simple Text Completion

The most basic usage involves sending a prompt and receiving a text response:

```yaml
- step: HUB_COMPLETION
  behavior: 'You are a helpful assistant.'
  instruction: 'Write a short summary of the benefits of unit testing. Answer in JSON format.'
  options:
    temperature: 0.2
  response_format: 'json_object'
```

### Using Stored Prompts

You can reference pre-configured prompts from the Prompt Management UI:

```yaml
- step: OPENAI_COMPLETION
  promptId: d112a306-8f7a-43bc-8f38-94161b6a91cd
```

## Configuration Options

### Core Prompts

| Option        | Type   | Description                                                          |
| ------------- | ------ | -------------------------------------------------------------------- |
| `behavior`    | string | System prompt that defines the AI's behavior and role                |
| `instruction` | string | User prompt containing the actual task or question                   |
| `promptId`    | string | ID of a stored prompt configuration (overrides behavior/instruction) |

Both `behavior` and `instruction` support template variables:

```yaml
- step: HUB_COMPLETION
  behavior: 'You are an expert newsroom assistant.'
  instruction: 'Summarize this article: {textContent}'
```

### Model Selection

| Option  | Type   | Description                                     |
| ------- | ------ | ----------------------------------------------- |
| `model` | string | Model ID to use (e.g., `gpt-4o`, `gpt-4-turbo`) |

For Azure deployments, the `deploymentId` from the service configuration is used instead.

### API Options

Configure model behavior through the `options` object:

```yaml
- step: OPENAI_COMPLETION
  instruction: 'Generate creative headlines'
  options:
    temperature: 0.9
    top_p: 0.95
    max_tokens: 150
    n: 3 # Generate 3 completions
```

Common options include:

- `temperature` - Controls randomness (0.0 = deterministic, 2.0 = very creative)
- `top_p` - Nucleus sampling parameter
- `max_tokens` - Maximum tokens in the response
- `frequency_penalty` - Reduces repetition of tokens
- `presence_penalty` - Encourages topic diversity

## Multimodal Inputs

### Image Inputs

Send images along with text prompts for vision tasks:

#### Single Image

```yaml
- step: HUB_COMPLETION
  instruction: "Describe what's in this image"
  image: '{{ flowContext.blob }}' # Blob or data URL
  imageDetail: 'high' # low, high, or auto
```

#### Multiple Images

```yaml
- step: HUB_COMPLETION
  instruction: 'Compare these images'
  images:
    - '{{ flowContext.image1 }}'
    - '{{ flowContext.image2 }}'
  imageDetail: 'auto'
```

Images can be provided as:

- Blob objects
- Data URLs (base64-encoded)
- Objects with `url` and `detail` properties

### Audio Inputs

Send audio files for transcription or analysis:

#### Single Audio File

```yaml
- step: HUB_COMPLETION
  instruction: 'Analyze the tone and content'
  audio: '{{ flowContext.audioBlob }}'
  audioFormat: 'wav' # or mp3, etc.
```

#### Multiple Audio Files

```yaml
- step: HUB_COMPLETION
  instruction: 'Compare these recordings'
  audios:
    - '{{ flowContext.audio1 }}'
    - '{{ flowContext.audio2 }}'
```

Audio can be provided as:

- Blob objects
- Data URLs
- Objects with `data` and `format` properties

### Generic Attachments

For more flexible multimodal inputs:

```yaml
- step: HUB_COMPLETION
  instruction: 'Process these attachments'
  attachments:
    - type: text
      text: 'Additional context'
    - type: image_url
      image_url:
        url: 'data:image/png;base64,...'
    - '{{ flowContext.someBlob }}' # Auto-detected type
```

## Conversation History

### Prior Messages

Include conversation context:

```yaml
- step: HUB_COMPLETION
  instruction: 'Continue our discussion'
  messages:
    - role: system
      content: 'You are a helpful assistant'
    - role: user
      content: 'What is AI?'
    - role: assistant
      content: 'AI stands for Artificial Intelligence...'
    - role: user
      content: 'Tell me more about neural networks'
```

## Structured Outputs

### JSON Object Mode

Request responses in JSON format:

```yaml
- step: HUB_COMPLETION
  instruction: 'List the top 3 benefits of automation. Answer in JSON format.'
  response_format: 'json_object'
```

The LLM will return valid JSON. Access the parsed object via `flowContext.object`.

### JSON Schema Mode

Define an exact schema for the response:

```yaml
- step: HUB_COMPLETION
  instruction: 'Extract person information from the text'
  response_format:
    type: json_schema
    json_schema:
      name: person
      schema:
        type: object
        properties:
          name:
            type: string
          age:
            type: number
          occupation:
            type: string
        required: ['name']
        additionalProperties: false
      strict: true
```

The parsed response is available in `flowContext.object`.

### List Mode

A convenient shorthand for generating lists:

```yaml
- step: HUB_COMPLETION
  instruction: 'Summarize in 3-5 key points: {textContent}'
  response_format: 'list'

- step: INSERT_LIST
  at: CURSOR
```

The list is available in `flowContext.list` as an array of strings.

## Tool Calling (Function Calling)

Enable the AI to call functions and interact with your system:

### Basic Tool Definition

```yaml
- step: HUB_COMPLETION
  behavior: 'You are a helpful assistant'
  instruction: 'Ask the user for their favorite color using askUser'
  tools:
    - name: askUser
      description: 'Ask the user a question'
      required_params: ['question']
      steps:
        - step: USER_PROMPT
          promptText: '{{ flowContext.question }}'
```

### Full Tool Schema

For more complex parameter validation:

```yaml
- step: HUB_COMPLETION
  instruction: 'Update the document metadata'
  tools:
    - name: updateMetadata
      description: 'Updates document metadata fields'
      parameters:
        type: object
        properties:
          title:
            type: string
            description: 'Document title'
          category:
            type: string
            enum: ['news', 'opinion', 'feature']
          tags:
            type: array
            items:
              type: string
        required: ['title']
        additionalProperties: false
      steps:
        - step: SET
          metadata:
            title: '{{ flowContext.title }}'
            category: '{{ flowContext.category }}'
```

### Tool Implementation Options

Tools can be implemented in three ways:

#### 1. Steps (Workflow)

```yaml
tools:
  - name: getArticleText
    description: 'Retrieves the full article text'
    steps:
      - step: SET
        articleContent: '{{ client.getTextContent() }}'
```

#### 2. Template (Simple Expression)

```yaml
tools:
  - name: getHeadline
    description: 'Gets the current headline'
    template: "{{ client.getTextAtXpath('/doc/story/headline') }}"
```

#### 3. Script (JavaScript)

```yaml
tools:
  - name: setHeadline
    description: 'Updates the document headline'
    required_params: ['newHeadline']
    script: |
      console.log("Updating headline to:", params.newHeadline);
      if (params?.newHeadline) {
        client.replaceTextAtXpath(
          params.newHeadline,
          '/doc/story/headline'
        );
      }
      return { success: true };
```

### Model Context Protocol (MCP) Tools

You can enable built-in tools from Model Context Protocol (MCP) servers connected via the Hub.

```yaml
- step: HUB_COMPLETION
  builtinTools:
    # Enable all tools from 'filesystem' MCP server
    - mcp: 'filesystem'

    # Enable specific tools only
    - mcp: 'github'
      only: ['create_issue', 'list_repos']

    # Create a semantic alias with preset arguments
    - alias: 'saveReport'
      target: 'mcp__filesystem__write_file'
      description: 'Save the generated report'
      args:
        path: '/documents/report.md'
```

The MCP tools are automatically fetched from the configured Hub server and exposed to the LLM. The `mcp` property specifies the server name as registered in the Hub.

**Shorthand names for MCP tools**  
The `only` list and alias targets accept several shorthand formats:

- `<tool>` (e.g. `read_file`)
- `mcp__<server>__<tool>`
- `mcp.<server>.<tool>`
- `<server>__<tool>`
- `<server>.<tool>`

### Calling MCP Tools Directly

You can also call MCP tools directly as a step in your workflow, without going through an LLM. This is useful for deterministic operations or when you know exactly which tool to call.

```yaml
- step: HUB_MCP_INVOKE
  server: filesystem
  tool: read_file
  arguments:
    path: '/documents/report.md'
```

The result of the tool call is stored in `flowContext.object` (or `flowContext.text` depending on the result type).

### Listing Available MCP Tools

To list all available tools for a specific MCP server:

```yaml
- step: HUB_MCP_TOOLS
  server: filesystem
```

The result is stored in `flowContext.object` as a list of tool definitions.

### Tool Aliases

You can define custom "virtual tools" that wrap built-in tools with preset arguments. This simplifies the tool definition for the AI and ensures consistent parameter usage.

For a complete list of available tools, see the [Built-in AI Tools](./builtin-ai-tools.md) reference.

```yaml
builtinTools:
  - alias: getHeadline
    target: getTextAtXpath
    description: 'Returns the main headline'
    args:
      xpath: '/doc/story/headline'
```

Alias argument behavior:

- Arguments defined in `args` are fixed defaults.
- Fixed top-level arguments are removed from the tool schema sent to the LLM.
- Remaining parameters are still exposed and must be supplied by the agent.
- With `toolStrict: true`, attempts to override fixed top-level arguments are rejected.

#### Tool Alias Templates

If you need multiple aliases with shared defaults, define a template and extend it:

```yaml
builtinTools:
  - aliasTemplate: markSegments
    target: highlightTextSegments
    description: 'Mark segments with consistent matching behavior'
    args:
      tag: 'mark'
      virtual: true
      match:
        normalizeCharacters: true
        normalizeWhitespace: true
        caseSensitive: false
        mode: 'first'

  - alias: markClaims
    extend: markSegments
    description: 'Mark claims for fact checking'
    args:
      attributes:
        class: 'fact-check'
```

### Dynamic Built-in Tool Discovery

Use `autoDiscoverTools: true` when you want the model to discover and activate built-in tool providers on demand.

When enabled, two meta-tools are exposed:

- `listAvailableTools` - Returns discoverable providers and their tool schemas.
- `activateTools` - Activates selected providers for the current completion loop.

```yaml
- step: HUB_COMPLETION
  autoDiscoverTools: true
  instruction: |
    First call listAvailableTools to inspect what is available.
    Then call activateTools for only the providers you need.
    After activation, call the provider tools directly.
```

You can combine auto-discovery with explicit `builtinTools` for aliases, MCP tools, or always-on tools:

```yaml
- step: HUB_COMPLETION
  autoDiscoverTools: true
  builtinTools:
    - alias: getHeadline
      target: getTextAtXpath
      args:
        xpath: '/doc/story/headline'
```

Auto-discovery respects provider-specific conditions. For example, `FeedbackTools` and `UserInteractionTools` are discoverable only when the Flow Execution Monitor is enabled.

### User Interaction Tools

ProActions provides specialized built-in tools for prompting users during AI workflows. These tools display interactive UI in the Flow Execution Monitor and pause workflow execution until the user responds.

When `autoDiscoverTools` is enabled, these tools are listed and activatable only if the monitor is active.

#### Available Tools

| Tool | Purpose | Returns |
|------|---------|---------|
| `askSingleChoice` | User selects ONE option from buttons | `{ answerId, answerLabel, customInput? }` |
| `askMultipleChoice` | User checks MULTIPLE options (checkboxes) | `{ selectedIds[], selectedLabels[] }` |
| `askFreeformQuestion` | User types free text | `{ answer }` |
| `askConfirmation` | Simple Yes/No question | `{ confirmed }` |

#### Basic Usage Examples

```yaml
- step: HUB_COMPLETION
  behavior: 'You are a newsroom assistant'
  instruction: 'Help classify and improve this article'
  builtinTools:
    - askSingleChoice
    - askConfirmation
  tools:
    # AI can now call these tools directly
```

**AI Agent Example Calls:**
```javascript
// Single choice
askSingleChoice("Select article category", [
  { id: "news", label: "📰 News" },
  { id: "sports", label: "⚽ Sports" },
  { id: "opinion", label: "💭 Opinion" }
])

// Multiple choice with checkboxes
askMultipleChoice("Select relevant tags", [
  { id: "breaking", label: "Breaking News" },
  { id: "politics", label: "Politics" },
  { id: "local", label: "Local" }
], 1, 3)  // Min 1, Max 3 selections

// Simple confirmation
askConfirmation("Publish this article to production?")
```

#### Advanced Features: askSingleChoice

The `askSingleChoice` tool supports advanced options for custom input and skip functionality:

| Parameter | Type | Description |
|-----------|------|-------------|
| `allowCustomInput` | boolean | Shows text field below buttons for custom user input |
| `allowSkip` | boolean | Automatically adds "Skip" button |
| `skipLabel` | string | Custom label for skip button (default: "Skip") |

**Examples:**

```javascript
// With skip option
askSingleChoice("Choose template", templates, false, true, "No template")
// Returns: { answerId: "__skip__", answerLabel: "No template" }

// With custom input
askSingleChoice("Select author", suggestedAuthors, true)
// User can click button AND type custom text
// Returns: { answerId: "john", answerLabel: "John", customInput: "Associate Editor" }
```

#### Alias Override Patterns

Use aliases to create reusable, preconfigured tools with controlled parameter flexibility:

**Pattern 1: Full Override (All Fixed)**

AI cannot change any parameters:

```yaml
builtinTools:
  - alias: confirm_delete
    target: askConfirmation
    args:
      question: "Are you sure you want to permanently delete this?"
```

Usage: `confirm_delete()` - No arguments allowed

**Pattern 2: Partial Override (Mixed Control)**

Fix some parameters, allow AI to customize others:

```yaml
builtinTools:
  - alias: select_priority
    target: askSingleChoice
    args:
      # question NOT specified - AI can customize
      answers:  # Fixed choices
        - id: "urgent"
          label: "🔴 Urgent"
        - id: "high"
          label: "🟠 High Priority"
        - id: "normal"
          label: "🟢 Normal"
      allowSkip: true  # Fixed
      skipLabel: "Keep current priority"
```

Usage: `select_priority("What priority?")` - AI provides question, rest is fixed

**Pattern 3: Smart Defaults with Flexibility**

Provide sensible defaults but allow complete customization:

```yaml
builtinTools:
  - alias: choose_author_with_suggestions
    target: askSingleChoice
    args:
      question: "Select or enter author name"
      answers:  # Fixed suggestions
        - id: "john_smith"
          label: "John Smith"
        - id: "jane_doe"
          label: "Jane Doe"
      allowCustomInput: true  # Always allow custom
      # AI can override question if needed
```

**Pattern 4: Contextual Question Templates**

Let AI inject context into questions:

```yaml
builtinTools:
  - alias: confirm_publish_action
    target: askSingleChoice
    args:
      # AI provides full dynamic question
      answers:
        - id: "proceed"
          label: "✓ Proceed"
          description: "Continue with this action"
        - id: "cancel"
          label: "✗ Cancel"
      allowSkip: false  # Force decision
```

Usage: `confirm_publish_action("Publish to Production site?")` 

**Pattern 5: Category/Tag Selection with Custom Input**

```yaml
builtinTools:
  - alias: tag_article
    target: askMultipleChoice
    args:
      question: "Select article tags (choose multiple)"
      answers:
        - id: "breaking"
          label: "Breaking News"
          description: "Time-sensitive content"
        - id: "analysis"
          label: "Analysis"
        - id: "opinion"
          label: "Opinion Piece"
        - id: "local"
          label: "Local News"
      minSelect: 1
      maxSelect: 3
      # AI cannot override these constraints
```

#### Alias Override Rules

| What you specify in `args` | Effect |
|---------------------------|---------|
| **Parameter present** | Fixed value - AI cannot override |
| **Parameter absent** | Dynamic - AI can provide value |
| **With `toolStrict: true`** | Attempts to override fixed params are rejected |

**Best Practice:** Only fix parameters you absolutely need to control. Leave others unspecified for maximum AI flexibility.

#### Return Value Handling

All user interaction tools return JSON strings. Parse them in workflows:

```yaml
- step: HUB_COMPLETION
  instruction: 'Ask user to select priority and update metadata'
  builtinTools:
    - alias: choose_priority
      target: askSingleChoice
      args:
        answers:
          - id: "high"
            label: "High"
          - id: "low"
            label: "Low"

# AI calls: choose_priority("Select priority")
# Returns: '{"answerId":"high","answerLabel":"High"}'

- step: SCRIPTING
  script: |
    const result = JSON.parse(flowContext.text);
    flowContext.priority = result.answerId;
```


### Tool Calling Options

| Option                      | Type    | Description                                                          |
| --------------------------- | ------- | -------------------------------------------------------------------- |
| `autoDiscoverTools`         | boolean | Enable dynamic discovery via `listAvailableTools` and `activateTools` meta-tools |
| `toolChoice`                | string  | Control tool selection policy: `auto`, `required`, `none` (mapped to `tool_choice`) |
| `parallelToolCalls`         | boolean | Allow multiple tool calls in one assistant turn (mapped to `parallel_tool_calls`) |
| `maxToolIterations`         | number  | Maximum rounds of tool calls (default: 5)                            |
| `afterToolResultToolChoice` | string  | Behavior after tool execution: `auto` or `none`                      |
| `toolsReuseContext`         | boolean | Whether to pass full flowContext to tool steps/templates (preferred) |
| `functionsReuseContext`     | boolean | Deprecated alias of `toolsReuseContext`                              |
| `toolErrorBehavior`         | string  | How to handle tool/function errors: `retry` (default), `fail`, or `once` |
| `functionErrorBehavior`     | string  | Deprecated alias of `toolErrorBehavior`                              |
| `toolGuidance`              | object  | Optional tool schema guidance (e.g., table columns or metric keys)   |
| `toolStrict`                | boolean | Enforce strict tool argument validation (missing/extra args fail)    |

`toolChoice` and `parallelToolCalls` are camelCase in YAML and are translated to API fields.

Prefer `toolErrorBehavior` and `toolsReuseContext` in new flows. The `function*` variants remain for backward compatibility.

For advanced forcing of a specific tool object, pass `tool_choice` via `options`.

### Tool Guidance (Tables & Metrics)

Use `toolGuidance` to tighten tool schemas so the LLM sends the right structure:

```yaml
- step: HUB_COMPLETION
  toolGuidance:
    addTableFeedback:
      columns: ['Claim', 'Verdict', 'Source']
      description: 'Fact-check results'
    addMetricsFeedback:
      metrics: ['engagement', 'clarity', 'specificity', 'seo']
      min: 0
      max: 100
```

This updates the tool schema the model sees, ensuring it sends the exact column and metric keys.

### Tool Guardrails (Strict Mode)

Enable `toolStrict` to enforce required fields and reject extra arguments:

```yaml
- step: HUB_COMPLETION
  toolStrict: true
```

This prevents silent failures when the model sends incorrect parameters.

### Managing Tool Call Errors

Control how the system handles tool execution errors to prevent infinite retry loops:

```yaml
- step: HUB_COMPLETION
  behavior: 'You are a helpful assistant with access to various tools.'
  instruction: 'Help the user complete their task.'
  toolErrorBehavior: 'once' # Prevent retrying failed tools
  maxToolIterations: 3
  tools:
    - name: processData
      description: 'Process user data'
      required_params: ['data']
      steps:
        - step: SCRIPTING
          script: |
            if (!flowContext.data) {
              throw new Error("Data parameter is required");
            }
            flowContext.processed = flowContext.data.toUpperCase();
            return flowContext;
```

**Error Behavior Modes:**

- **`retry`** (default) - Allows the LLM to retry failed tools. Use for recoverable errors where different parameters might succeed.
- **`fail`** - Stops the entire flow immediately when any tool fails. Use for critical operations where errors should halt execution.
- **`once`** - Prevents retrying the same tool after it fails once. Recommended to prevent infinite loops when a tool has persistent errors.

When a tool fails:

- With `retry`: The LLM receives the error and may retry with different parameters
- With `fail`: The flow stops with a clear error message
- With `once`: The LLM is informed the tool cannot be retried and should try a different approach

**Best Practice:** Use `toolErrorBehavior: "once"` for production workflows to prevent wasted API calls from repeatedly failing tools.

### Example: Interactive Workflow

```yaml
- step: HUB_COMPLETION
  behavior: |
    You are an expert newsroom assistant.
    You help editors refine articles by analyzing content and improving headlines.
  instruction: |
    1. Use getTextContent to retrieve the article.
    2. Ask the user for their preferred headline style using askUser.
    3. Create an improved headline based on the article and user feedback.
    4. Use setHeadline to update the document.
    5. Confirm completion.
  tools:
    - name: askUser
      description: 'Ask the user a question'
      required_params: ['question']
      steps:
        - step: USER_PROMPT
          promptText: '{{ flowContext.question }}'

    - name: getTextContent
      description: 'Returns the full document text'
      template: '{{ client.getTextContent() }}'

    - name: setHeadline
      description: 'Updates the document headline'
      required_params: ['newHeadline']
      script: |
        if (params?.newHeadline) {
          client.replaceTextAtXpath(
            params.newHeadline,
            '/doc/story/headline'
          );
        }
```

## Audio Output

Request audio responses from the model:

```yaml
- step: HUB_COMPLETION
  instruction: 'Explain quantum computing in simple terms'
  outputAudio:
    voice: 'alloy' # alloy, echo, fable, onyx, nova, shimmer
    format: 'mp3' # mp3, opus, aac, flac, wav, pcm
  model: 'gpt-4o-audio-preview'
```

Outputs:

- `flowContext.audio` - Audio data URL
- `flowContext.audio_transcript` - Text transcript (if provided)

### Combined Text and Audio Output

```yaml
- step: HUB_COMPLETION
  behavior: 'Create an audio description of the image'
  instruction: '{{ flowContext.text }}'
  image: '{{ flowContext.blob }}'
  outputAudio:
    voice: alloy
    format: mp3
  model: gpt-4o-mini-audio-preview-2024-12-17

- step: PLAY_AUDIO
  in: '{{ flowContext.audio }}'
```

## Reasoning Models

For models that support reasoning (e.g., o1, o3):

```yaml
- step: HUB_COMPLETION
  instruction: 'Solve this complex problem: ...'
  options:
    reasoning:
      effort: 'low' # Model-specific reasoning configuration
  model: 'o1'
```

The reasoning output (if provided) is available in `flowContext.reasoning`.

## Streaming

Enable streaming mode to receive tokens incrementally as they are generated, rather than waiting for the complete response. Streamed tokens are displayed in real time in the Flow Execution Monitor.

### Basic Streaming

```yaml
- step: HUB_COMPLETION
  behavior: 'You are a creative writing assistant.'
  instruction: 'Write a short story about a robot discovering nature.'
  stream: true
  options:
    temperature: 0.8
```

### Streaming Options

| Option     | Type    | Default | Description                                                                                  |
| ---------- | ------- | ------- | -------------------------------------------------------------------------------------------- |
| `stream`   | boolean | `false` | Enable streaming mode for incremental token delivery                                         |
| `feedback` | boolean | `true`  | Show streaming tokens in the Flow Execution Monitor. Set to `false` for silent streaming      |

### Streaming with Tool Calls

Streaming works seamlessly with tool calling. During tool-call-only iterations (where the model invokes tools without producing text), no feedback entry is created in the monitor — only text responses are streamed to the UI.

```yaml
- step: HUB_COMPLETION
  behavior: 'You are a headline optimization assistant.'
  instruction: |
    1. Read the current headline using getHeadline.
    2. Propose an improved headline and confirm with the user.
    3. If confirmed, update it with setHeadline.
  stream: true
  toolChoice: required
  toolStrict: true
  maxToolIterations: 10
  builtinTools:
    - addFeedback
    - askConfirmation
    - alias: getHeadline
      target: getTextAtXpath
      description: 'Returns the article headline'
      args:
        xpath: '/doc/story/grouphead/headline/p'
    - alias: setHeadline
      extend: getHeadline
      target: replaceTextAtXpath
      description: 'Changes the headline'
```

### Silent Streaming (Performance Only)

Use `feedback: false` to benefit from streaming's faster time-to-first-token without displaying tokens in the monitor:

```yaml
- step: HUB_COMPLETION
  instruction: 'Summarize the article: {textContent}'
  stream: true
  feedback: false
```

### Automatic Fallback

Streaming is automatically disabled for configurations that are incompatible with it. No changes are needed — the step silently falls back to non-streaming:

- **`response_format: list`** or **`json_schema`** — requires the SDK's `.parse()` method
- **`outputAudio`** — audio output is binary, not streamable as text
- **`n > 1`** — multiple choices interleave chunks from different completions

### Proxy Configuration for Streaming

Streaming requires that all proxies between the browser and the API pass Server-Sent Events (SSE) without buffering. If tokens arrive all at once instead of incrementally, check your proxy configuration:

```nginx
location /swing/proactions/ {
    proxy_buffering off;
    proxy_cache off;
    gzip off;
    brotli off;  # or: proxy_set_header Accept-Encoding "";

    # existing proxy_pass configuration...
}
```

The critical settings are `proxy_buffering off` and disabling compression (`gzip off`, `brotli off`) on the SSE location. Compression algorithms buffer the response body before sending, which defeats streaming.

## Outputs

The completion steps produce various outputs in `flowContext`:

| Output             | Type   | Description                                             |
| ------------------ | ------ | ------------------------------------------------------- |
| `text`             | string | Main text response (always available)                   |
| `object`           | object | Parsed JSON object (when using `response_format`)       |
| `list`             | array  | Array of strings (when using `response_format: "list"`) |
| `audio`            | string | Audio data URL (when `outputAudio` is configured)       |
| `audio_transcript` | string | Transcript of audio response                            |
| `response`         | object | Full API response object                                |
| `reasoning`        | object | Reasoning data from models that support it              |
| `choices`          | array  | All response choices (if `n > 1`)                       |
| `usage`            | object | Token usage statistics                                  |
| `tool_results`     | array  | Results from tool/function calls                        |

## Service Configuration

Configure services in your `services.yaml`:

```yaml
HUB:
  type: completion
  endpoint: '{BASE_URL}/swing/proactions'
  target: openai
  defaultBehavior: 'You are a helpful assistant for journalists.'
  options:
    temperature: 0.7
    max_tokens: 800

OPENAI_COMPLETION:
  type: completion
  apiKey: sk-...
  endpoint: 'https://api.openai.com/v1'
  model: gpt-4o
  options:
    temperature: 0.7

AZURE_OPENAI_COMPLETION:
  type: completion
  endpoint: 'https://your-resource.openai.azure.com'
  apiKey: '...'
  deploymentId: 'gpt-4o'
  apiVersion: '2024-02-15-preview'
```

## Practical Examples

### Vision: Image Caption Generation

```yaml
- step: SELECTED_OBJECT

- step: EDAPI_OBJECT_CONTENT
  format: lowres

- step: HUB_COMPLETION
  behavior: |
    You are a professional photo editor. Create short, precise,
    factual captions suitable for news articles.
  instruction: 'Create a caption for this image. Article: {textContent}'
  image: '{{ flowContext.blob }}'

- step: REPLACE_TEXT
  at: XPATH
  xpath: '/doc/story/text'
```

### Generate Structured Data

```yaml
- step: HUB_COMPLETION
  instruction: 'Extract key information from: {textContent}'
  response_format:
    type: json_schema
    json_schema:
      name: article_metadata
      schema:
        type: object
        properties:
          title:
            type: string
          keywords:
            type: array
            items:
              type: string
          summary:
            type: string
        required: ['title', 'keywords']
        additionalProperties: false
      strict: true
```

### Interactive Table Generation

```yaml
- step: HUB_COMPLETION
  behavior: |
    Transform plain text into structured HTML tables.
    Use only: table, tr, td, b, i, u tags.
    Output valid XML with no blank lines.
  instruction: 'Convert to table: {selectedText}'

- step: SANITIZE
  stripMarkdown: true
  validateAndRepairXml: true

- step: INSERT_XML
  at: CURSOR
  position: insertBefore
```

### Multi-Modal Analysis

```yaml
- step: SELECTED_OBJECT

- step: EDAPI_OBJECT_CONTENT
  format: lowres

- step: HUB_COMPLETION
  behavior: 'You are an accessibility expert.'
  instruction: |
    Create a detailed description for visually impaired users.
    Include all visible elements, text, context, and meaning.
  image: '{{ flowContext.blob }}'

- step: HUB_COMPLETION
  behavior: 'Create an audio version of this description.'
  instruction: '{{ flowContext.text }}'
  outputAudio:
    voice: alloy
    format: mp3
  model: gpt-4o-mini-audio-preview-2024-12-17

- step: PLAY_AUDIO
  in: '{{ flowContext.audio }}'
```

## Best Practices

### 1. Use Appropriate Models

- Use smaller/faster models (`gpt-4o-mini`) for simple tasks
- Use larger models (`gpt-4o`, `gpt-4-turbo`) for complex reasoning
- Use specialized models for audio (`gpt-4o-audio-preview`) or vision tasks

### 2. Optimize Prompts

- Be specific and clear in instructions
- Provide context in the `behavior` (system prompt)
- Use examples when needed
- Request specific output formats

### 3. Handle Errors

Always wrap completion calls in error handling:

```yaml
- step: TRY
  steps:
    - step: HUB_COMPLETION
      instruction: '...'
  catch:
    - step: SHOW_NOTIFICATION
      notificationType: error
      message: 'AI processing failed. Please try again.'
```

### 4. Manage Costs

- Set `max_tokens` to limit response length
- Use `temperature: 0` for deterministic outputs
- Store frequently used prompts with `promptId`
- Use tool calling to reduce back-and-forth iterations

### 5. Structured Outputs

- Always use `response_format` when you need JSON
- Use `strict: true` in schemas for guaranteed adherence
- Validate outputs - especially XML - with the `SANITIZE` step

### 6. Image Processing

- Use `imageDetail: "low"` for faster processing when detail isn't critical
- Use `imageDetail: "high"` for detailed analysis
- Compress images before sending to reduce costs

### 7. Conversation Context

- Limit message history to recent exchanges
- Clean up old messages to reduce token usage
- Store important context in `flowContext` variables

## Security Considerations

### API Key Protection

Never hardcode API keys in YAML files:

```yaml
# Bad
OPENAI_COMPLETION:
  apiKey: sk-actual-key-here

# Good - use environment variables or proxy
HUB:
  endpoint: '{BASE_URL}/swing/proactions'
  target: openai
```

### Content Filtering

Consider implementing content filters for user-facing applications:

```yaml
- step: HUB_COMPLETION
  safetyIdentifier: 'user-generated-content'
  instruction: '...'
```

## Troubleshooting

### Common Issues

**Issue**: "No completion message returned"

- Check that the service configuration is correct
- Verify API credentials
- Check network connectivity

**Issue**: Tool calls exceed max iterations

- Increase `maxToolIterations` if legitimate iterations are needed
- Use `toolErrorBehavior: "once"` to prevent retrying failed tools
- Use `toolErrorBehavior: "fail"` to stop immediately on errors
- Simplify tool descriptions to help the LLM choose correctly
- Use `afterToolResultToolChoice: "none"` to stop after first tool call

**Issue**: JSON parsing errors

- Use `response_format: "json_object"` or structured schemas
- Validate output with `SANITIZE` step
- Check that instructions explicitly request JSON

**Issue**: Image not recognized

- Verify image is in supported format (JPEG, PNG, GIF, WebP)
- Check image size limits
- Ensure data URL is properly formatted

## Performance Tips

1. **Enable Streaming**: Use `stream: true` for long responses — users see output immediately instead of waiting for the full generation
2. **Batch Operations**: Process multiple items in one call when possible
3. **Parallel Processing**: Use `PARALLEL` step for independent completions
4. **Model Selection**: Choose the smallest model that meets your needs

## Limits and Constraints

- **Maximum Tokens**: Varies by model (typically 4K-128K input tokens)
- **Image Size**: Maximum ~20MB per image
- **Audio Size**: Maximum ~25MB per audio file
- **Tool Iterations**: Default 5, configurable up to reasonable limits
- **Rate Limits**: Depend on service tier and provider

## Further Reading

- [Service Configuration Guide](./service-configuration.md)
- [Workflow Design Best Practices](./workflow-design.md)
- [Template Variables and Expressions](../howto/use-nunjucks-templates.md)
