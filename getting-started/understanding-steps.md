# Understanding Steps

Steps are the building blocks of ProActions workflows. Each step performs a specific operation, and steps are executed sequentially in the order they appear in your flow.

## Step Anatomy

Every step in ProActions follows this basic structure:

```yaml
- step: STEP_NAME
  # Step-specific configuration options
  option1: value1
  option2: value2
```

### Common Step Properties

- **step**: The type of step to execute (required)
- Configuration properties vary by step type (refer to step documentation for specifics)

## Step Categories

ProActions provides steps organized into several categories:

### Control Flow Steps

Control the execution flow of your workflow:

- **IF**: Execute steps conditionally based on a condition
- **FOR**: Loop through a fixed number of iterations or array items
- **WHILE**: Repeat steps while a condition is true
- **SWITCH**: Choose between multiple execution paths
- **TRY**: Handle errors gracefully with try/catch logic
- **BREAK**: Break out of loops
- **END** / **END_IF**: Stop flow execution conditionally (aliases)
- **SET**: Assign values to variables
- **CALL_TEMPLATE** / **CALL_FLOW**: Reuse predefined step templates or scoped flows (aliases)
- **EMIT_EVENT** / **DISPATCH_EVENT**: Emit custom events for action subscriptions (aliases)
- **PLATFORM**: Route execution to platform-specific branches (swing/prime/standalone)
- **PARALLEL**: Run multiple step lists in parallel

**Example:**

```yaml
- step: IF
  condition: '{{ flowContext.selectedText }}'
  then:
  - step: SHOW_NOTIFICATION
    message: 'You have selected text!'
  else:
  - step: SHOW_NOTIFICATION
    message: 'No text selected'
```

### LLM Steps

Interact with Large Language Models and AI services:

- **HUB_COMPLETION**: Chat/completion via ProActions Hub (recommended)
- **HUB_TRANSCRIPTION**: Audio-to-text transcription
- **HUB_SPEECH**: Text-to-speech generation
- **HUB_IMAGE_GENERATION**: Generate images with DALL·E
- **HUB_CONTENT_EXTRACTION**: Extract content from URLs
- **OPENAI_COMPLETION**: Direct OpenAI integration
- **OPENAI_ASSISTANT**: Use OpenAI Assistants
- **STABILITY_AI_UPSCALE**: Upscale images
- **STABILITY_AI_SEARCH_AND_REPLACE**: Edit images with AI

**Example:**

```yaml
- step: HUB_COMPLETION
  behavior: 'You are a helpful assistant for journalists.'
  instruction: 'Summarize this article: {textContent}'
  options:
    max_tokens: 150
```

### Editor Steps

Interact with the document editor:

- **GET_TEXT_CONTENT**: Retrieve text from the document
- **GET_XML_CONTENT**: Retrieve XML content
- **INSERT_TEXT**: Insert text at a location
- **REPLACE_TEXT**: Replace text at a location
- **INSERT_XML**: Insert XML content
- **REPLACE_XML**: Replace XML content
- **SELECTED_OBJECT**: Get info about selected content object
- **SET_METADATA**: Update metadata fields
- **CLEAR_SELECTION**: Clear text selection

**Example:**

```yaml
- step: GET_TEXT_CONTENT
  at: XPATH
  xpath: '/doc/story/body'

- step: REPLACE_TEXT
  at: CURSOR_PARAGRAPH
```

### User Interaction Steps

Display UI elements and collect user input:

- **FORM**: Display a custom form to collect input
- **PROMPT**: Show a simple text input prompt
- **USER_SELECT**: Show a selection list
- **IMAGE_PICKER**: Let users pick from a list of images
- **FILE_UPLOAD**: Upload files from the user's system
- **SHOW_NOTIFICATION**: Display notifications
- **SHOW_RESPONSE**: Show content in a modal
- **SHOW_PROGRESS**: Display a progress bar
- **UPDATE_PROGRESS**: Update progress bar state
- **HIDE_PROGRESS**: Hide the progress bar

**Example:**

```yaml
- step: PROMPT
  promptText: 'Enter the article title:'

- step: SHOW_NOTIFICATION
  message: 'Processing complete!'
```

### Service Steps

Integrate with external services:

- **REST**: Make generic REST API calls
- **DEEPL_TRANSLATE**: Translate text with DeepL
- **DEEPL_WRITE**: Rephrase text with DeepL
- **ELEVENLABS_TTS**: Text-to-speech with ElevenLabs
- **ELEVENLABS_STT**: Speech-to-text with ElevenLabs
- **BRIGHTER_AI**: Anonymize images (faces, license plates)

**Example:**

```yaml
- step: DEEPL_TRANSLATE
  instruction: '{selectedText}'
  target_lang: 'FR'
```

### EDAPI Steps

Work with Eidosmedia content objects:

- **UPLOAD**: Upload content to EDAPI
- **UPLOAD_IMAGE**: Upload images
- **UPDATE_BINARY_CONTENT**: Update existing binary content
- **EDAPI_OBJECT_CONTENT**: Fetch object content

### Utility Steps

Data transformation and manipulation:

- **TO_LIST**: Convert text to a list
- **PARSE_JSON**: Parse JSON strings
- **MARKDOWN_TO_HTML**: Convert markdown to HTML
- **SANITIZE**: Clean and validate text/XML
- **BASE64_TO_BLOB**: Convert base64 to blob
- **SCRIPTING**: Execute custom JavaScript

**Example:**

```yaml
- step: SET
  text: |
    1. First item
    2. Second item
    3. Third item
- step: TO_LIST
```

### Browser Steps

Interact with the browser:

- **DOWNLOAD**: Download files to the user's system
- **READ_CLIPBOARD**: Read from clipboard
- **WRITE_CLIPBOARD**: Write to clipboard

**Example:**

```yaml
- step: SET
  text: '{{ flowContext.generatedContent }}'
- step: WRITE_CLIPBOARD
```

## Step Execution Flow

Steps execute in sequence, and each step can:

1. **Read inputs** from the flow context or built-in variables
2. **Process data** according to its configuration
3. **Write outputs** back to the flow context for use by subsequent steps

### Data Flow Example

```yaml
- step: GET_TEXT_CONTENT
  at: CURSOR

- step: HUB_COMPLETION
  instruction: 'Summarize: {{ flowContext.text }}'

- step: SHOW_RESPONSE
  message: 'Summary complete'
```

## Default Inputs and Outputs

Many steps automatically pass data between each other through the flow context:

- Steps that produce text output make it available to the next step
- Steps that consume text input can use the output from the previous step
- Each step implementation determines what it reads and writes

This allows for concise syntax:

```yaml
# The completion result is automatically available to REPLACE_TEXT
- step: HUB_COMPLETION
  instruction: 'Translate to French: {selectedText}'

- step: REPLACE_TEXT
  at: CURSOR
```

## Conditional Execution with IF

Use the `IF` step to execute steps conditionally:

```yaml
- step: IF
  condition: '{{ client.isTextSelected() }}'
  then:
    - step: SHOW_NOTIFICATION
      message: 'Processing your selection'
  else:
    - step: SHOW_NOTIFICATION
      message: 'Please select text first'
```

## Error Handling

Use the `TRY` step to handle errors gracefully:

```yaml
- step: TRY
  try:
    - step: HUB_COMPLETION
      instruction: 'Process this: {textContent}'
  catch:
    - step: SHOW_NOTIFICATION
      message: 'An error occurred. Please try again.'
```

## Step Templates

You can define reusable step templates and call them:

```yaml
# In your configuration
TEMPLATES:
  translateToFrench:
    - step: DEEPL_TRANSLATE
      target_lang: "FR"


# In your flow
- step: CALL_TEMPLATE
  name: translateToFrench
```

## Reusable Blocks and Scoped References

Reusable blocks let you define constants, vars, flows, and scripts once and reference them by scope.

```yaml
AI_KIT:
  BLOCKS:
    constants:
      modulesBasePath: /SysConfig/ProActions/lib
    flows:
      normalizeStory:
        - step: HUB_COMPLETION
          instruction: 'Normalize this article: {textContent}'

  SECTIONS:
    - section: Editorial
      imports:
        - from: '{{ global.constants.modulesBasePath }}/common.ai-kit.module.yaml'
          as: common
      blocks:
        vars:
          defaultFlowRef: global.flows.normalizeStory
      actions:
        - title: Normalize Story
          blocks:
            scripts:
              sumFromParams: params.a + params.b
          flow:
            - step: CALL_FLOW
              ref: '{{ section.vars.defaultFlowRef }}'
            - step: SCRIPTING
              scriptRef: local.scripts.sumFromParams
              returnAs: sum
```

### Migration from `TEMPLATES`

- Existing `TEMPLATES` + `CALL_TEMPLATE.name` flows continue to work.
- You can migrate gradually by introducing `BLOCKS.flows` and `CALL_FLOW.ref` for new reusable flows.
- `CALL_TEMPLATE` also supports `ref`, so old and new styles can coexist during migration.

### Import Resolution Notes

- Section imports are available to all actions in that section.
- Action imports are merged with section imports.
- If both define the same alias, the action alias overrides for that action scope.
- Duplicate aliases in the same imports list are invalid.

## Working with Inputs and Outputs

Some steps allow you to customize how they read inputs and write outputs using the `inputs` and `outputs` configuration:

```yaml
- step: SELECTED_OBJECT
  outputs:
    - type: object
      name: selectedObject # will store output in flowContext.selectedObject

- step: EDAPI_OBJECT_CONTENT
  inputs:
    - type: text
      name: 'selectedObject.id' # will read value from flowContext.selectedObject.id
```

## Next Steps

- Learn about [Working with Variables](./working-with-variables.md) to master data flow
- Explore [Using Services](./using-services.md) to integrate external APIs
- Check the complete [Step Library](../step-library/overview.md) for all available steps
