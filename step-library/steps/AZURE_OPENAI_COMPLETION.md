---
sidebar_position: 1
---

# AZURE_OPENAI_COMPLETION

Chat/completion step that calls OpenAI (or Hub/Azure variants). Supports messages, images, audio, attachments, tools (functions), structured outputs (json_schema/json_object/list), reasoning and audio output. Many step-level configuration options are supported via cfg.*.

:::info Documentation Inheritance
Documentation partly inherited from `OPENAI_COMPLETION`.
:::

## At a glance
- **Category** LLM
- **Aliases** SERVICE_AZURE_OPENAI_COMPLETION
- **Version:** 1.0.0
- **Applications:** all
- **Scope:** all
- **Default Service:** AZURE_OPENAI_COMPLETION

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
- step: AZURE_OPENAI_COMPLETION
  behavior: "You are a helpful assistant."
  instruction: "Write a short summary of the benefits of unit testing. Answer in JSON format."
  options:
    temperature: 0.2
  response_format: "json_object"
```

### Use a stored prompt from Prompt Management UI.
```yaml
  - step: AZURE_OPENAI_COMPLETION
    promptId: d112a306-8f7a-43bc-8f38-94161b6a91cd
```

### Use structured response with predefined type "list"
```yaml
- step: AZURE_OPENAI_COMPLETION
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
- step: AZURE_OPENAI_COMPLETION
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
- step: AZURE_OPENAI_COMPLETION
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
- step: AZURE_OPENAI_COMPLETION
  instruction: "Calculate 5 + 3"
  tools:
    - name: calculateSum
      description: Calculate the sum of two numbers
      required_params: ["a", "b"]
      scriptRef: local.scripts.sumFromParams
```

### Enable streaming for real-time response display in Flow Monitor
```yaml
- step: AZURE_OPENAI_COMPLETION
  behavior: "You are a creative writing assistant."
  instruction: "Write a short story about a robot discovering nature for the first time."
  stream: true  # Tokens are displayed incrementally in the Flow Monitor
  options:
    temperature: 0.8
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
