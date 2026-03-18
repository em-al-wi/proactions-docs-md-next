---
sidebar_position: 1
---

# OPENAI_COMPLETION

Chat/completion step that calls OpenAI (or Hub/Azure variants). Supports messages, images, audio, attachments, tools (functions), structured outputs (json_schema/json_object/list), reasoning and audio output. Many step-level configuration options are supported via cfg.*.

## At a glance
- **Category** LLM
- **Aliases** SERVICE_OPENAI_COMPLETION
- **Version:** 1.0.0
- **Applications:** all
- **Scope:** all
- **Default Service:** OPENAI_COMPLETION

## When to Use


Use **OPENAI_COMPLETION** when you need to leverage OpenAI models for text generation, analysis, or transformation.

**Use OPENAI_COMPLETION when:**
- You need text generation, completion, or transformation with OpenAI compatible APIs
- You want to analyze or extract structured information from text
- You need JSON/structured output from natural language input
- You want to use function/tool calling capabilities
- You need to process images with vision models (gpt-4-vision, gpt-4o)
- You need to process or generate audio (with supported models)

**Don't use OPENAI_COMPLETION when:**
- You only need simple text templating without AI - use **SET** with \`\{\{ \}\}\` syntax instead
- You need translation specifically - use **DEEPL_TRANSLATE** for better quality
- You need local processing without external API calls - use **SCRIPTING** instead
  

## Prerequisites

**Required Services:**
- OPENAI_COMPLETION service must be configured in services.yaml
- Valid OpenAI API key in service configuration or proxy

**Required Context:**
- flowContext.text (optional) - Input text for processing; if not provided, uses editor content or instruction templates
- flowContext.messages (optional) - Conversation history to continue from previous interactions

**Platform Requirements:**
- Works on all platforms (Swing, Prime)
- Requires internet connectivity for API calls

## Data Flow


**Input Sources:**
1. \`cfg.instruction\` - User prompt (supports templates like \`\{\{ flowContext.variable \}\}\`)
2. \`cfg.behavior\` - System prompt to set AI behavior and personality
3. \`flowContext.text\` - Default input text (used when instruction contains templates)
4. \`cfg.images\` / \`cfg.image\` - Optional image inputs (requires vision-capable model)
5. \`cfg.audio\` / \`cfg.audios\` - Optional audio inputs (requires audio-capable model)
6. \`cfg.attachments\` - Generic attachments (images, audio, or text files)
7. \`cfg.messages\` - Full conversation history (overrides flowContext.messages)
8. \`cfg.tools\` - Tool/function definitions for function calling (\`cfg.functions\` also supported for backward compatibility)

**Processing Flow:**
1. Resolves templates in instruction and behavior prompts
2. Loads stored prompt if \`promptId\` is specified
3. Builds messages array (system message + user message)
4. Attaches images/audio/attachments if provided
5. Calls OpenAI API with configured model and options
6. Handles tool/function calls if tools are defined (iterative execution)
7. Processes response based on \`response_format\` setting
8. Extracts and formats output according to response type

**Output Destinations:**
- \`flowContext.text\` - Text response (default output, always set)
- \`flowContext.object\` - Structured object (when \`response_format\` is "json_object" or json_schema)
- \`flowContext.list\` - Array of items (when \`response_format\` is "list")
- \`flowContext.audio\` - Audio data URL (when \`outputAudio\` is configured)
- \`flowContext.audio_transcript\` - Audio transcript text (optional)
- \`flowContext.reasoning\` - Reasoning trace (for reasoning-capable models)
- \`flowContext.response\` - Full API response object (optional, for debugging)
- \`flowContext.usage\` - Token usage statistics (optional)
- \`flowContext.choices\` - Raw choices array from API (optional)
- \`flowContext.tool_results\` - Results from function/tool executions (optional)
  

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `model` | Model id or deployment id to use | None |false| false |None|None|
| `promptId` | Id of a stored prompt configuration to reuse | None |false| false |None|None|
| `instruction` | The user prompt to send to the LLM (supports templates) | None |false| true |None|None|
| `behavior` | The system prompt to send to the LLM (supports templates) | None |false| false |None|None|
| `options` | Additional request options object (temperature, n, top_p, model, etc.) | None |false| false |None|None|
| `response_format` | Structured response format. Can be a string ("json_object" or "list") or an object with json_schema definition | None |false| false |None|None|
| `outputAudio` | Configuration object to request audio output (voice/format) | None |false| false |None|None|
| `toolChoice` | Optional tool selection policy (mapped to tool_choice in the model request) | None |false| false |None|None|
| `parallelToolCalls` | Allow model/providers to emit multiple tool calls in one assistant turn (mapped to parallel_tool_calls) | None |false| false |None|None|
| `maxToolIterations` | Maximum number of tool iterations to perform | None |false| false |None|None|
| `afterToolResultToolChoice` | Behavior after a tool result (none\|auto) | None |false| false |None|None|
| `toolErrorBehavior` | How to handle tool execution errors: "retry" allows LLM to retry failed tools (default), "fail" stops the flow immediately on any tool error, "once" prevents retrying the same tool after it fails once | None |false| false |None|None|
| `functionErrorBehavior` | How to handle function execution errors (deprecated, use toolErrorBehavior instead) | None |false| false |None|None|
| `tools` | Array of tool/function descriptors for tool calling. Each descriptor can define template, steps, script, or scriptRef. | None |false| false |None|None|
| `functions` | Array of function descriptors for tool calling (deprecated, use tools instead). Supports template, steps, script, or scriptRef. | None |false| false |None|None|
| `toolsReuseContext` | Whether to reuse full flowContext when executing tool templates | None |false| false |None|None|
| `functionsReuseContext` | Whether to reuse full flowContext when executing function templates (deprecated, use toolsReuseContext instead) | None |false| false |None|None|
| `safetyIdentifier` | Optional safety identifier for the request | None |false| false |None|None|
| `messages` | Array of prior messages to include in the conversation (overrides flowContext.messages) | None |false| false |None|None|
| `images` | Array or single image input(s) defined inline via cfg.images or cfg.image | None |false| false |None|None|
| `image` | Single inline image config | None |false| false |None|None|
| `audio` | Single inline audio config | None |false| false |None|None|
| `audios` | Array of audio inputs | None |false| false |None|None|
| `reasoning` | Reasoning configuration object to pass to the API (for models supporting reasoning) | None |false| false |None|None|
| `imageDetail` | Image detail level for image inputs (low, high, auto) | auto |false| false |None|None|
| `audioFormat` | Audio format for audio inputs (e.g., wav, mp3) | None |false| false |None|None|
| `attachments` | Array of generic attachments (images, audio, text) to include in the user message | None |false| false |None|None|
| `autoDiscoverTools` | 'Enable dynamic tool discovery. When true, the agent can discover and activate tool providers on demand ' +
    'via listAvailableTools and activateTools meta-tools, instead of requiring all tools to be listed explicitly. ' +
    'Can be combined with builtinTools for aliases, MCP tools, and explicit provider configurations.' | false |false| false |None|None|
| `builtinTools` | List of built-in tool classes to enable for the AI model | None |false| false |None|None|
| `toolGuidance` | Optional tool schema guidance (e.g., addTableFeedback columns or addMetricsFeedback metric keys and ranges) | None |false| false |None|None|
| `toolStrict` | Enable strict tool argument validation (reject missing/extra arguments) | false |false| false |None|None|
| `stream` | Enable streaming mode for incremental response delivery. When enabled, tokens are delivered as they are generated rather than waiting for the full response. Streaming is automatically disabled for incompatible configurations (json_schema, list, audio output). Defaults to false. | false |false| false |None|None|
| `feedback` | Show streaming response in the Flow Execution Monitor. Defaults to true when stream is enabled. Set to false to use streaming for performance (faster time-to-first-token) without monitor visualization. | None |false| false |None|None|
| `skills` | 'List of skill names to load. Skills provide system instructions and reference documents. ' +
    'Each skill is loaded from `/SysConfig/ProActions/Skills/<skillName>/SKILL.md`.' | None |false| false |None|None|

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `text` | Default textual response (setTextOutput) from the completion | false |
| `object` | Structured object output when using json/object response formats | false |
| `list` | List output extracted from structured response (response_format=list) | false |
| `audio` | Audio data URL or blob when requesting audio output | false |
| `audio_transcript` | Optional transcript produced as part of audio response | false |
| `response` | Raw API response object saved as optional output | false |
| `reasoning` | Optional reasoning object provided by the model/SDK | false |
| `choices` | Optional choices array (raw) from the completion | false |
| `usage` | Optional usage object from the completion result | false |
| `tool_results` | Optional array of tool results produced during execution | false |

## Examples

### Simple completion.
```yaml
- step: OPENAI_COMPLETION
  behavior: "You are a helpful assistant."
  instruction: "Write a short summary of the benefits of unit testing. Answer in JSON format."
  options:
    temperature: 0.2
  response_format: "json_object"
```

### Use a stored prompt from Prompt Management UI.
```yaml
  - step: OPENAI_COMPLETION
    promptId: d112a306-8f7a-43bc-8f38-94161b6a91cd
```

### Use structured response with predefined type "list"
```yaml
- step: OPENAI_COMPLETION
  instruction:
    "Summary in 3-5 key points. Max. 100 characters per entry. Precise
    presentation of the top information. Focus on the core statements of the
    article. Favour clear and concrete information. Answers in list form. Article text: {{ client.getTextContent() }}"
  response_format: "list" # use structured data
- step: INSERT_LIST
  at: CURSOR
```

### Use tools/function calls to interact with an LLM.
```yaml
- step: OPENAI_COMPLETION
  behavior: |
    You are an expert newsroom assistant.
  instruction: |
    Ask the user what to put into the headline using the function askHeadline.
    Then generate a nice headline based on the user's input.
  tools:
    - name: askHeadline
      description: Ask the user what to write into the headline
      required_params: ["question"]
      steps:
        - step: USER_PROMPT
          promptText: "{{ flowContext.question }}"
```

### Control function error retry behavior
```yaml
- step: OPENAI_COMPLETION
  behavior: "You are a helpful assistant with access to various tools."
  instruction: "Help the user complete their task using available functions."
  functionErrorBehavior: "once"  # Don't retry failed functions - prevents loops
  maxToolIterations: 3
  tools:
    - name: processData
      description: Process user data
      required_params: ["data"]
      steps:
        - step: SCRIPTING
          script: |
            if (!params.data) {
              throw new Error("Data parameter is required");
            }
            flowContext.processed = params.data.toUpperCase();
            return flowContext;
```

### Use scriptRef directly in a tool definition
```yaml
- step: OPENAI_COMPLETION
  instruction: "Calculate 5 + 3"
  tools:
    - name: calculateSum
      description: Calculate the sum of two numbers
      required_params: ["a", "b"]
      scriptRef: local.scripts.sumFromParams
```

### Enable streaming for real-time response display in Flow Monitor
```yaml
- step: OPENAI_COMPLETION
  behavior: "You are a creative writing assistant."
  instruction: "Write a short story about a robot discovering nature for the first time."
  stream: true  # Tokens are displayed incrementally in the Flow Monitor
  options:
    temperature: 0.8
```

## Common Pitfalls

:::warning Don't use high temperature for factual or structured output tasks

**Why:** High temperature (> 0.7) increases randomness, which can produce creative but inaccurate or inconsistent responses, especially problematic for JSON output or factual extraction

**Solution:** Use temperature 0.0-0.3 for factual tasks, summaries, structured output, or data extraction. Use 0.7-1.0 only for creative writing, brainstorming, or varied responses.
:::

:::warning Don't forget to handle API failures and empty responses

**Why:** API calls can fail due to rate limits, network issues, invalid API keys, or service outages. Empty responses can break subsequent steps that expect data.

**Solution:** Wrap OPENAI_COMPLETION in TRY block for error handling, and validate output exists before using it in subsequent steps.
:::

:::warning Don't include sensitive or confidential data in prompts without data processing agreement

**Why:** Data sent to OpenAI's API may be stored and potentially used for model training unless you have a specific data processing agreement.

**Solution:** Use Azure OpenAI with private deployment for sensitive data, or sanitize/anonymize information before sending. Review OpenAI data usage policies.
:::

## Performance Notes


**API Call Timing:**
- Simple completions (< 1k tokens): 1-3 seconds
- Large context (10k+ tokens): 3-10 seconds
- Function calling: +1-2 seconds per tool call iteration
- Vision (image analysis): +2-5 seconds depending on image size

**Cost Optimization:**
- Use cheaper models (gpt-3.5-turbo, gpt-4o-mini) for simple tasks
- Set \`max_tokens\` in options to prevent unexpectedly long (expensive) responses
- Cache results when processing identical content multiple times
- Use \`response_format\` for structured output instead of parsing free-form text
  

## See Also

**Related Steps:**
- **[AZURE_OPENAI_COMPLETION](AZURE_OPENAI_COMPLETION.md)** - Alternative for Azure deployments: Use AZURE_OPENAI_COMPLETION when you have Azure-hosted OpenAI models. Same functionality but uses Azure endpoints and authentication.
- **[HUB_COMPLETION](HUB_COMPLETION.md)** - Alternative using ProActions Hub: Use HUB_COMPLETION for centralized API key management and request routing through ProActions Hub middleware.

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
