# Step Library Overview

ProActions provides a comprehensive library of workflow steps organized into categories. Each step performs a specific function and can be combined to create powerful automation workflows.

## Quick Reference

Browse steps by category:


### Browser


- **[DOWNLOAD](./steps/DOWNLOAD.md)** - Downloads a file constructed from text or an existing Blob. The filename is required.

- **[READ_CLIPBOARD](./steps/READ_CLIPBOARD.md)** - Reads content from the user clipboard. Prefers HTML content when available, falls back to plain text.

- **[WRITE_CLIPBOARD](./steps/WRITE_CLIPBOARD.md)** - Writes text to the user clipboard. Reads the content from the configured input (cfg.text) or from the default text input in the flow context.



### Control


- **[BREAK](./steps/BREAK.md)** - Breaks out of the current loop (e.g., WHILE or FOR) based on the given condition.

- **[CALL_TEMPLATE](./steps/CALL_TEMPLATE.md)** - Calls a reusable flow by legacy template name (`name`) or scoped reference (`ref`).

- **[DEBUG](./steps/DEBUG.md)** - Logs debug information about the current flow context and step configuration. Optionally triggers a debugger breakpoint.

- **[EMIT_EVENT](./steps/EMIT_EVENT.md)** - Emits a custom internal event that can trigger other actions subscribed via action events.

- **[END](./steps/END.md)** - Ends the execution of a flow based on the given condition, with optional notification.

- **[FOR](./steps/FOR.md)** - Executes a set of steps for a fixed number of iterations (numeric) or iterates over an array (items).

- **[IF](./steps/IF.md)** - Execute a conditional branch based on a condition

- **[PARALLEL](./steps/PARALLEL.md)** - Runs multiple step lists in parallel and merges their flow contexts upon completion.

- **[PLATFORM](./steps/PLATFORM.md)** - Routes flow execution to platform-specific branches (`swing`, `prime`, `standalone`) with optional `default` fallback.

- **[SET](./steps/SET.md)** - Sets variables in the flow context. Can execute steps, evaluate expressions, or set key-value pairs.

- **[SLEEP](./steps/SLEEP.md)** - Pause execution for a given delay (ms). Useful for pacing flows or waiting for external state changes.

- **[SWITCH](./steps/SWITCH.md)** - Evaluates a switch value and executes the steps for the matching case. Supports both array and object syntax for cases.

- **[TRY](./steps/TRY.md)** - Executes a set of steps in a try block, and if an error occurs, executes catch steps.

- **[WHILE](./steps/WHILE.md)** - Repeatedly executes a set of steps while a condition evaluates to true.



### EDAPI


- **[EDAPI_OBJECT_CONTENT](./steps/EDAPI_OBJECT_CONTENT.md)** - Fetch object content from EDAPI and return it as blob or configured outputs. The content id can be provided via cfg.contentId or via a text input.

- **[UPDATE_BINARY_CONTENT](./steps/UPDATE_BINARY_CONTENT.md)** - Update binary content of an existing object in EDAPI using the provided blob. Returns updated object metadata.

- **[UPLOAD](./steps/UPLOAD.md)** - Upload content to EDAPI by providing a blob or specifying parameters (filename, type). Returns created object metadata.

- **[UPLOAD_IMAGE](./steps/UPLOAD_IMAGE.md)** - Upload an image to EDAPI by providing an image object (url or blob). Returns the created object metadata as output.



### Editor


- **[CLEAR_SELECTION](./steps/CLEAR_SELECTION.md)** - Clears the current text selection and optionally repositions the cursor to the start or end of the selection. Available in Swing editor context only.

- **[CLEAR_TAGS](./steps/CLEAR_TAGS.md)** - Removes all occurrences of the specified tags from the document or report while preserving their text content. This is useful for cleaning up temporary markup or removing specific tag types.

- **[CLOSE_DOCUMENT](./steps/CLOSE_DOCUMENT.md)** - Closes the current document without saving.

- **[GET_PURE_TEXT](./steps/GET_PURE_TEXT.md)** - Retrieves the full pure text content from the current document and sets it as output.

- **[GET_TEXT_CONTENT](./steps/GET_TEXT_CONTENT.md)** - Retrieves text content from the document or a specific location and sets it as output.

- **[GET_XML_CONTENT](./steps/GET_XML_CONTENT.md)** - Retrieves XML content from the document or a specific location and sets it as output.

- **[GET_ZONES](./steps/GET_ZONES.md)** - Retrieves zones and their linked documents from a DWP or Report.

- **[INSERT_LIST](./steps/INSERT_LIST.md)** - Insert a list of items into the document. The items can be provided as a list input or be generated from a text input (converted to a list using ProcessToList).

- **[INSERT_OBJECT](./steps/INSERT_OBJECT.md)** - Insert object references or templates into the current document using a DTX template.

Prime: Supports paths, UUIDs, and LOIDs (auto-formatted). Options insertionType, full, and templateType are ignored (warnings logged).

- **[INSERT_TEXT](./steps/INSERT_TEXT.md)** - Insert text at the given location (cursor, xpath or report). If the step reads text from the flowContext, you can use a preceding SET step to provide the content.

- **[INSERT_XML](./steps/INSERT_XML.md)** - Insert XML content at the given location. The content can be provided via the flowContext (using `in`) or via the default text input.

- **[REPLACE_TEXT](./steps/REPLACE_TEXT.md)** - Replace existing text at the given location (cursor paragraph, cursor or xpath) with the provided content. Content can come from a flow variable (using `in`) or the default text input.

- **[REPLACE_XML](./steps/REPLACE_XML.md)** - Replace XML content at the given location. Content can be sourced from a flow variable (`in`) or the default text input.

- **[SAVE_DOCUMENT](./steps/SAVE_DOCUMENT.md)** - Saves the current document. Optionally closes the document after saving.

- **[SELECTED_OBJECT](./steps/SELECTED_OBJECT.md)** - Retrieves information about the currently selected content object and sets it as output.

- **[SET_METADATA](./steps/SET_METADATA.md)** - Set a field value in the object panel using a selector or XPath. Supports tagsinput and plain fields.

- **[TAG_PURE_TEXT](./steps/TAG_PURE_TEXT.md)** - Adds tags to pure text content in the document. Tags can highlight or annotate specific text ranges. In Swing, tags can be virtual (temporary) or persistent. In Prime, tags are always persistent.



### LLM


- **[AZURE_OPENAI_COMPLETION](./steps/AZURE_OPENAI_COMPLETION.md)** - Chat/completion step that calls OpenAI (or Hub/Azure variants). Supports messages, images, audio, attachments, tools (functions), structured outputs (json_schema/json_object/list), reasoning and audio output. Many step-level configuration options are supported via cfg.*.

- **[AZURE_OPENAI_IMAGE_GENERATION](./steps/AZURE_OPENAI_IMAGE_GENERATION.md)** - ImageGeneration using Azure OpenAI.

- **[AZURE_OPENAI_SPEECH](./steps/AZURE_OPENAI_SPEECH.md)** - Azure OpenAI speech step alias.

- **[AZURE_OPENAI_TRANSCRIPTION](./steps/AZURE_OPENAI_TRANSCRIPTION.md)** - Azure OpenAI transcription alias (delegates to the OpenAI transcription implementation).

- **[HUB_COMPLETION](./steps/HUB_COMPLETION.md)** - Chat/completion step that calls OpenAI (or Hub/Azure variants). Supports messages, images, audio, attachments, tools (functions), structured outputs (json_schema/json_object/list), reasoning and audio output. Many step-level configuration options are supported via cfg.*.

- **[HUB_CONTENT_EXTRACTION](./steps/HUB_CONTENT_EXTRACTION.md)** - Extract textual content from a URL using the ProActions-Hub extraction tool. The URL can be provided via the default text input or the `in`/`url` configuration.

- **[HUB_IMAGE_GENERATION](./steps/HUB_IMAGE_GENERATION.md)** - ImageGeneration using ProActions Hub.

- **[HUB_SPEECH](./steps/HUB_SPEECH.md)** - ProActionsHub-compatible speech generation step. Inherits behavior from OPENAI_SPEECH.

- **[HUB_TRANSCRIPTION](./steps/HUB_TRANSCRIPTION.md)** - Hub-compatible transcription step (delegates to hub service). Writes transcription text and optional segments list.

- **[OPENAI_ASSISTANT](./steps/OPENAI_ASSISTANT.md)** - Create or reuse an OpenAI assistant. If cfg.assistantId is provided the assistant will be retrieved; otherwise a new assistant is created. The created or retrieved assistant is written to the flow context.

- **[OPENAI_COMPLETION](./steps/OPENAI_COMPLETION.md)** - Chat/completion step that calls OpenAI (or Hub/Azure variants). Supports messages, images, audio, attachments, tools (functions), structured outputs (json_schema/json_object/list), reasoning and audio output. Many step-level configuration options are supported via cfg.*.

- **[OPENAI_DELETE_THREAD](./steps/OPENAI_DELETE_THREAD.md)** - Delete a thread using its id. Provide the thread object or id via inputs.

- **[OPENAI_IMAGE_GENERATION](./steps/OPENAI_IMAGE_GENERATION.md)** - Generate images using OpenAI image models (DALL·E and GPT-Image-1). Supports model, size, quality, count (n), response_format and other parameters. The generated images are stored in the configured output key or default `imageList`.

- **[OPENAI_SPEECH](./steps/OPENAI_SPEECH.md)** - Generate speech audio using the OpenAI Speech API (or corresponding Hub/Azure variants). Reads text from the default text input and returns an audio blob.

- **[OPENAI_THREAD](./steps/OPENAI_THREAD.md)** - Create or reuse an OpenAI thread. If cfg.reuse and cfg.storeIn are provided the thread id may be reused. The created thread is written to the flow context.

- **[OPENAI_THREAD_FILES](./steps/OPENAI_THREAD_FILES.md)** - Create a vector store for thread file tools and optionally upload files. Writes created store to flow context and optional uploaded file list to an optional output.

- **[OPENAI_THREAD_MESSAGE](./steps/OPENAI_THREAD_MESSAGE.md)** - Send a message to a thread/assistant. The assistant and thread inputs are required. The message can be provided via cfg.instruction or via default text input. The assistant response text is stored in the default text output.

- **[OPENAI_TRANSCRIPTION](./steps/OPENAI_TRANSCRIPTION.md)** - Transcribe audio using OpenAI (or Azure/Hub variants). Reads an audio file/blob from inputs and writes the transcription text and optional segment list.

- **[STABILITY_AI_OUTPAINT](./steps/STABILITY_AI_OUTPAINT.md)** - Outpaint an image region using StabilityAI. Supports coordinates (left,right,up,down) to define the outpainting area.

- **[STABILITY_AI_SEARCH_AND_RECOLOR](./steps/STABILITY_AI_SEARCH_AND_RECOLOR.md)** - Search and recolor elements in an image using StabilityAI. Provide select_prompt and prompt to find and recolor elements.

- **[STABILITY_AI_SEARCH_AND_REPLACE](./steps/STABILITY_AI_SEARCH_AND_REPLACE.md)** - Search and replace regions or objects within an image using StabilityAI. Provide a prompt and a search_prompt to locate and replace visual elements.

- **[STABILITY_AI_UPSCALE](./steps/STABILITY_AI_UPSCALE.md)** - Upscale an image using StabilityAI upscale endpoint. Uploads a provided image blob and returns the upscaled image blob.



### Service


- **[BRIGHTER_AI](./steps/BRIGHTER_AI.md)** - Upload an image to BrighterAI, poll for processing completion and download the anonymized image. The resulting anonymized image blob is written to the configured output.

- **[DEEPL_TRANSLATE](./steps/DEEPL_TRANSLATE.md)** - Translate text using DeepL. Supports many optional DeepL parameters and returns the translated text as the default text output.

- **[DEEPL_WRITE](./steps/DEEPL_WRITE.md)** - Rephrase/write text using DeepL Write endpoint. Returns the rephrased text in the default text output.

- **[ELEVENLABS_STT](./steps/ELEVENLABS_STT.md)** - Transcribe audio using ElevenLabs STT. Provides the transcribed text and optional JSON result.

- **[ELEVENLABS_TTS](./steps/ELEVENLABS_TTS.md)** - Generate speech audio from text using ElevenLabs TTS. Text is taken from the default text input or a flow variable via `in`.

- **[HUB_MCP_INVOKE](./steps/HUB_MCP_INVOKE.md)** - Invoke a tool on an MCP server via the Hub.

- **[HUB_MCP_TOOLS](./steps/HUB_MCP_TOOLS.md)** - List available tools from an MCP server via the Hub.

- **[REST](./steps/REST.md)** - Generic REST step. Builds a Request from the given configuration (url, method, headers, parameters, body or formData) and executes it using the host HTTP helper. Results are written to cfg.outputs when provided or to the default text output.



### UI


- **[CHANGE_VIEW_SIZE](./steps/CHANGE_VIEW_SIZE.md)** - Requests a change of the view port size. Useful to resize the command palette view in Prime 8.

- **[FILE_UPLOAD](./steps/FILE_UPLOAD.md)** - Opens a file upload dialog (modal) and stores selected file(s) into the flow context. Supports output mapping via the `outputs` config or default file output.

- **[FORM](./steps/FORM.md)** - Displays a configurable form to the user (via FormBuilder). The form definition is provided in `cfg.form`. When submitted, the returned values are merged into the flowContext.

- **[HIDE_PROGRESS](./steps/HIDE_PROGRESS.md)** - Hide any shown progress bar.

- **[IMAGE_PICKER](./steps/IMAGE_PICKER.md)** - Shows an image picker modal allowing the user to pick one image from a provided list of image objects (\{ url, previewUrl?, label? \}). The selected image object is stored as an output.

- **[MONITOR_CHOICE](./steps/MONITOR_CHOICE.md)** - Display a selection of choices in the Flow Monitor. Pauses workflow until user selects.

- **[MONITOR_CONFIRM](./steps/MONITOR_CONFIRM.md)** - Display a confirmation dialog in the Flow Monitor. Pauses workflow until user responds.

- **[MONITOR_INPUT](./steps/MONITOR_INPUT.md)** - Display an inline text input field in the Flow Monitor.

- **[MONITOR_JSON](./steps/MONITOR_JSON.md)** - Display JSON structured data in the Flow Monitor.

- **[MONITOR_KEYVALUE](./steps/MONITOR_KEYVALUE.md)** - Display key-value structured data in the Flow Monitor.

- **[MONITOR_METRICS](./steps/MONITOR_METRICS.md)** - Display metrics in the Flow Monitor.

- **[MONITOR_STATUS](./steps/MONITOR_STATUS.md)** - Display status messages in the Flow Monitor during workflow execution.

- **[MONITOR_TABLE](./steps/MONITOR_TABLE.md)** - Display tabular structured data in the Flow Monitor.

- **[PLAY_AUDIO](./steps/PLAY_AUDIO.md)** - Plays audio in a lightweight floating player. Audio source can be provided as a data URL, http(s) URL, or raw base64 via `cfg.in` or via the default text input. The step mounts a closed Shadow DOM audio player and resolves when the player is closed by the user.

- **[PROMPT](./steps/PROMPT.md)** - Shows a simple prompt to the user and stores the entered text into the default text output.

- **[SHOW_NOTIFICATION](./steps/SHOW_NOTIFICATION.md)** - Shows a notification to the user with optional title and resolved message.

- **[SHOW_PROGRESS](./steps/SHOW_PROGRESS.md)** - Show a progress bar with optional status text. Use `position` to place it and `autoHide` to auto-hide after completion.

- **[SHOW_RESPONSE](./steps/SHOW_RESPONSE.md)** - Shows a response to the user in a modal. Can run inline steps before showing and supports HTML and Markdown modes.

- **[UPDATE_PROGRESS](./steps/UPDATE_PROGRESS.md)** - Update an existing progress bar. Provide percentage (0-100) and optional status text.

- **[USER_SELECT](./steps/USER_SELECT.md)** - Displays a selection modal allowing the user to pick one item from a list. The list may be provided directly or produced by a pre-processing step (inlineSteps). The selected value is written to the default text output.



### Utils


- **[BASE64_TO_BLOB](./steps/BASE64_TO_BLOB.md)** - Converts a base64-encoded string to a Blob and stores it in the flow context as a blob output.

- **[MARKDOWN_TO_HTML](./steps/MARKDOWN_TO_HTML.md)** - Converts markdown text to HTML and sets it as text output.

- **[PARSE_JSON](./steps/PARSE_JSON.md)** - Parses JSON content from the text input and stores the resulting object in the flow context.

- **[SANITIZE](./steps/SANITIZE.md)** - Sanitizes text input. Can strip markdown code blocks, normalize Unicode characters (quotes, dashes, spaces), and validate/repair XML content before forwarding the result to the flow context.

- **[SCRIPTING](./steps/SCRIPTING.md)** - Executes inline JavaScript (`script`) or a JS-style template (`template`). Use with caution — only in trusted flows.

- **[TO_LIST](./steps/TO_LIST.md)** - Convert a textual AI response into a list of strings. Detects JSON arrays, ordered/unordered lists, comma-separated values, or falls back to a single-item array.



### YouTube


- **[HUB_YOUTUBE_AUTH_INIT](./steps/HUB_YOUTUBE_AUTH_INIT.md)** - Initiate OAuth authorization flow for YouTube (requires ProActions-Hub). Opens the auth URL when available unless cfg.omitOpenAuth is true.

- **[HUB_YOUTUBE_AUTH_LOGOUT](./steps/HUB_YOUTUBE_AUTH_LOGOUT.md)** - Log out the configured Hub YouTube account. Uses POST to logout and returns the resulting object.

- **[HUB_YOUTUBE_AUTH_STATUS](./steps/HUB_YOUTUBE_AUTH_STATUS.md)** - Check the status of a previously initiated YouTube OAuth authorization flow.

- **[HUB_YOUTUBE_UPLOAD](./steps/HUB_YOUTUBE_UPLOAD.md)** - Upload a video to YouTube via the Hub service. Supports video file, optional thumbnail, metadata fields (title, description, privacyStatus, categoryId), tags and additionalMetadata. Progress can be shown when cfg.updateProgress is true.




## Using Steps

Steps are defined in YAML configuration files and executed as part of flows. Each step has:

- **Name**: The step identifier (e.g., `HUB_COMPLETION`, `REPLACE_TEXT`)
- **Configuration options**: Parameters that control the step's behavior
- **Inputs**: Data the step reads from the flow context
- **Outputs**: Data the step writes to the flow context

### Basic Example

```yaml
- step: HUB_COMPLETION
  behavior: "You are a helpful assistant"
  instruction: "Summarize this: {textContent}"
  outputs:
    - type: text
      name: summary
```

### Common Patterns

- **Control Flow**: Use IF, FOR, WHILE steps to control execution
- **Service Integration**: Call external APIs and AI services
- **Data Processing**: Transform and manipulate data
- **User Interaction**: Prompt users and display results

## Step Categories

Click on a category to see all available steps:

- [Control Flow](./control-steps.md) - Conditional logic, loops, error handling
- [LLM Services](./llm-steps.md) - AI completion, transcription, image generation
- [Service Integration](./service-steps.md) - External service integrations
- [UI Components](./ui-steps.md) - Forms, prompts, notifications, menus
- [Editor Operations](./editor-steps.md) - Text manipulation, content insertion
- [EDAPI](./edapi-steps.md) - Méthode object operations
- [Browser Functions](./browser-steps.md) - Clipboard, downloads
- [YouTube Integration](./youtube-steps.md) - YouTube API operations
- [Utilities](./utils-steps.md) - Helper functions and utilities

## Finding the Right Step

**Need to...?**

- **Make decisions?** → Use [IF](./steps/IF.md) or [SWITCH](./steps/SWITCH.md)
- **Loop over items?** → Use [FOR](./steps/FOR.md) or [WHILE](./steps/WHILE.md)
- **Call an AI service?** → Use [HUB_COMPLETION](./steps/HUB_COMPLETION.md)
- **Get user input?** → Use [FORM](./steps/FORM.md)
- **Transform data?** → Use [SCRIPTING](./steps/SCRIPTING.md)
- **Handle errors?** → Use [TRY](./steps/TRY.md)
- **Show results?** → Use [SHOW_RESPONSE](./steps/SHOW_RESPONSE.md) or [SHOW_NOTIFICATION](./steps/SHOW_NOTIFICATION.md)

## Version Information

This documentation is automatically generated from the ProActions source code and represents the current available steps in your installation.
