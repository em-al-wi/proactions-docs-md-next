---
sidebar_position: 100
---

# Release Notes

All notable changes to ProActions are documented in this file.

## Recent Highlights

### What's New in 1.2.0 (April, 2026)

- 📊 **Flow Monitor** - Real-time visualization and control of running workflows
- ⏱️ **Progress Tracking** - Percentage, step-based, and indeterminate progress indicators
- 🛑 **Cancellation Support** - Stop long-running flows with confirmation dialogs
- 💬 **Interactive Feedback** - Prompts and actions within the monitor
- 🔄 **Streaming Content** - Real-time LLM response display as it arrives
- 🎨 **Adaptive Display** - Compact pill for background work, full panel when attention is needed
- 📄 **Document Lifecycle Steps** - New `SAVE_DOCUMENT` and `CLOSE_DOCUMENT` steps with cross-platform support in Swing and Prime
- ⚡ **Event-Driven Actions** - New internal event system with action subscriptions and `EMIT_EVENT` / `DISPATCH_EVENT` for trigger-based automation
- 🤖 **Agent Skills** - Load reusable skill instructions from the server into AI completion steps
- 🛠️ **Built-in AI Tools** - Ready-made tools for document access, user interaction, and metadata; no custom function definitions required
- 🔌 **MCP Integration** - Call MCP server tools directly from flows with `HUB_MCP_TOOLS` and `HUB_MCP_INVOKE` steps
- 🏗️ **Block Drop Integration** - Trigger actions automatically when blocks are dropped in the Swing editor (`onBlockDrop`)
- 🧩 **New Control & Utility Steps** - `PLATFORM`, `GET_ZONES`, `GET_PURE_TEXT`, `TAG_PURE_TEXT`, `CLEAR_TAGS`
- ⚙️ **Section-Level Configuration** - Override `monitor` and other settings at section level
- 📡 **Monitor Interaction Steps** - Eight new `MONITOR_*` steps for interactive prompts and structured data display in the Flow Monitor
- 🗂️ **Reusable Blocks** - Define constants, variables, flows, and scripts in `BLOCKS` and share them across sections/actions with scoped namespaces
- 🔗 **CALL_FLOW / Scoped References** - New `CALL_FLOW` step alias and `ref` option to call flows defined in `BLOCKS` by dot-path
- 🔧 **Custom Template Filters** - Built-in Nunjucks filters for array/object traversal: `pluck`, `find`, `getAttr`, `compact`, `flatten`, `sum`

### What's New in 1.1.0 (latest)

- 🎨 **Interactive Diff** - Review and selectively accept/reject text changes with granular control
- ⚡ **Inline Steps** - Improved UX with background processing for FORM, USER_SELECT, and SHOW_RESPONSE
- 🔧 **Swing Integrations** - Native Swing toolbar, object action, and context menu support
- 🤖 **Enhanced Chat Completion** - Tool calling and improved multimodal support
- 🔄 **New Control Steps** - BREAK, FOR, TRY, and PARALLEL for advanced workflow logic
- 🎵 **Audio Playback** - New PLAY_AUDIO step with customizable player
- 📚 **Prompt Library** - Reuse stored prompts from Swing Prompt Management UI

### Major Features Since 1.0.0

- **Flow Monitor** (1.2.0) - Real-time workflow visualization and control
- **Agent Skills** (1.2.0) - Reusable AI instruction sets loaded from the server
- **Built-in AI Tools** (1.2.0) - Ready-made tools for AI function calling
- **MCP Integration** (1.2.0) - Call MCP tools directly from flows or expose to AI
- **Block Drop Integration** (1.2.0) - Trigger flows on drag-and-drop in Swing editor
- **Interactive Diff** (1.1.0) - Granular change review and selection
- **Swing Integration** (1.1.0) - Native inline toolbar and action integration
- **Schema Support** (1.0.11) - Auto-completion and validation for YAML configs
- **Form Enhancements** (1.0.10) - Rich form components (HTML, Markdown, Diff)
- **Progress Bars** (1.0.8) - Visual feedback for long-running workflows
- **Context Menus** (1.0.7) - Text selection and one-click menus
- **ProActions Hub** - Centralized API key management for 40+ AI providers
- **Multi-Language Support** - English, Spanish, French, Italian, German

---

## Version History

- [1.2.0 - April 23, 2026](#120---april-23-2026) - Flow Monitor, Agent Skills, Built-in AI Tools, MCP Integration, Block Drop
- [1.1.0 - October 24, 2025](#110---october-24-2025) - Interactive Diff, Swing Integration, Enhanced Workflows
- [1.0.12 - February 7, 2025](#1012---february-7-2025) - Slash Menu, ElevenLabs, Prime 8
- [1.0.11 - April 28, 2025](#1011---april-28-2025) - Schema Support, YouTube Integration
- [1.0.10 - February 27, 2025](#1010---february-27-2025) - Form Enhancements, Spanish Language
- [1.0.9 - January 26, 2025](#109---january-26-2025) - DeepL Write Integration
- [1.0.8 - December 13, 2024](#108---december-13-2024) - Progress Bars, Dashboard Actions
- [1.0.7 - November 14, 2024](#107---november-14-2024) - Context Menus, External Config
- [1.0.6 - October 11, 2024](#106---october-11-2024) - Permissions, DEBUG Step
- [1.0.5 - September 3, 2024](#105---september-3-2024) - Enhanced Metadata Access
- [1.0.4 - August 15, 2024](#104---august-15-2024) - Performance Improvements
- [1.0.3 - August 14, 2024](#103---august-14-2024) - Stability AI Integration

---

## Detailed Release Notes

### 1.2.0 - April 23, 2026

**📊 Flow Monitor**

A comprehensive monitoring system that provides real-time visualization and control of running ProActions workflows:

- **Real-time Visualization**: Monitor displays live progress, current step, and elapsed time for all running flows
- **Progress Tracking**: Multiple progress types supported:
  - Percentage-based (0-100%)
  - Step-based ("3 of 10 steps")
  - Indeterminate (spinner for unknown duration)
- **Cancellation Support**: Users can cancel long-running flows with optional confirmation dialogs
- **Interactive Feedback**: Display prompts, confirmations, and actions directly in the monitor
- **Streaming Content**: Real-time display of LLM streaming responses as they arrive
- **Adaptive Display**: Starts as a compact pill, expands to a full panel when user attention is needed:
  - Auto-expands for interactive feedback (prompts, confirmations, choices)
  - Stays as pill for transient status/debug messages
  - Completed flows fade after a configurable delay
  - Errors and summary content keep the pill visible until dismissed
- **Customization**: Full control over position, theme, visibility, and behavior
- **Permissions**: User/group/team-based access control
- **Multi-App Support**: Works seamlessly in both Swing and Prime applications
- **Keyboard Shortcuts**: Navigate flows and dismiss the monitor without leaving the keyboard
- **Auto-Scroll**: Automatically scrolls to the bottom of the monitor log as new messages arrive, with a dedicated scroll-to-bottom button

**Configuration:**

Global configuration:

```yaml
AI_KIT:
  MONITOR:
    enabled: true # Enable monitor globally
    position: bottom-right # Position on screen
    theme: auto # Visual theme (dark/light/auto)
    completionFadeDelay: 3000 # ms before done-pill fades
    maxVisible: 5 # Max concurrent executions shown
    apps: # Which apps to show monitor in
      - all # Options: swing, prime, all
    showProgress: true # Show progress bars
    showStepNames: true # Show current step name
    showElapsedTime: true # Show elapsed time
    enableCancellation: true # Allow cancelling flows
    confirmCancellation: true # Confirm before cancelling
    renderMarkdown: true # Render markdown in feedback
```

Action-level override:

```yaml
- title: 'Long Running Task'
  monitor:
    enabled: true
    showProgress: true
    confirmCancellation: false
  flow:
    - step: HUB_COMPLETION
      instruction: 'Generate comprehensive report...'
```

Section-level override:

```yaml
AI_KIT:
  SECTIONS:
    - section: 'My Section'
      monitor:
        enabled: false # Disable monitor for all actions in this section
      actions:
        - title: 'Quick Action'
          flow:
            - step: SHOW_NOTIFICATION
              message: 'Done'
```

**Use Cases:**

- **Long-running workflows**: Provide visibility into multi-step LLM operations
- **Batch processing**: Track progress through large datasets
- **Interactive flows**: Handle user prompts without interrupting execution
- **Debugging**: Understand workflow execution and identify bottlenecks
- **User training**: Demonstrate what ProActions is doing in real-time

See the [Flow Monitor Guide](./guides/configuration/flow-monitor.md) for complete documentation.

**⚡ Event-Driven Action Triggers**

- Actions can subscribe to internal events using `events` in action configuration.
- New `EMIT_EVENT` (`DISPATCH_EVENT`) step emits custom events for automation chains.
- Trigger policies support `skip`, `queue`, and `parallel` behavior while an action is running.
- Supported event sources: `aikit`, `system`, `custom`.

See the [Swing Integration Guide](./guides/configuration/swing-integration.md) for full documentation on event-driven triggers.

**🤖 Agent Skills**

Skills are reusable instruction sets stored on the server that can be dynamically loaded into AI completion steps at runtime. This separates prompt engineering from flow configuration and makes instruction sets reusable across multiple actions.

- Skills are stored as `SKILL.md` files in `/SysConfig/ProActions/Skills/<skillName>/`
- Reference documents (e.g., style guides, terminology lists) can be stored under `<skillName>/references/`
- When loaded, skills are injected into the system prompt automatically
- A built-in `read_skill_reference` tool is exposed to the AI so it can retrieve reference documents on demand
- The base path for skills is configurable via `skillsBasePath` in `pro-actions-config.js`

```yaml
- step: HUB_COMPLETION
  behavior: 'You are a writing assistant.'
  instruction: 'Improve this article: {textContent}'
  skills:
    - editorial-style    # Loads /SysConfig/ProActions/Skills/editorial-style/SKILL.md
    - seo-guidelines     # Loads /SysConfig/ProActions/Skills/seo-guidelines/SKILL.md
```

See the [AI Integration Guide — Agent Skills](./guides/configuration/ai-integration.md#agent-skills) for full documentation including loading modes, reference documents, and the base path configuration.

**🛠️ Built-in AI Tools**

ProActions now provides a library of pre-configured tools that can be enabled for AI function calling without writing custom function definitions. These tools give the LLM direct access to document content, metadata, and user interaction.

- **Content tools**: `getTextContent`, `getXmlContent`, `getTextAtXpath`, `replaceTextAtCursor`, `insertXmlAtCursor`, and many more
- **Document info tools**: `getDocumentId`, `getMetadata`, `getWorkflowStatus`, `getChannel`, etc.
- **Container tools**: For container (report/DWP) documents
- **User interaction tools**: `askSingleChoice`, `askMultipleChoice`, `askFreeformQuestion`, `askConfirmation` — pause the AI and prompt the user directly from the monitor
- **Tool aliases**: Define virtual tools with preset arguments to simplify what the AI sees
- **Alias templates**: Share configuration across multiple aliases using `aliasTemplate` / `extend`
- **Auto-discovery**: Use `autoDiscoverTools: true` to let the model discover and activate available tools on demand

```yaml
- step: HUB_COMPLETION
  instruction: 'Classify the article and update its metadata'
  builtinTools:
    - ContentTools
    - DocumentInfoTools
    - alias: getHeadline
      target: getTextAtXpath
      args:
        xpath: '/doc/story/headline'
```

See the [Built-in AI Tools Reference](./guides/configuration/builtin-ai-tools.md) for the complete tool catalog.

**🔌 MCP (Model Context Protocol) Integration**

Connect to MCP servers via ProActions Hub and expose their tools to LLMs or invoke them directly from flows.

- **`HUB_MCP_TOOLS`**: List all available tools from an MCP server
- **`HUB_MCP_INVOKE`**: Directly invoke an MCP tool from a flow step (without an LLM)
- Use `builtinTools` with `mcp:` entries to expose MCP tools to an AI completion step

```yaml
# Invoke an MCP tool directly
- step: HUB_MCP_INVOKE
  server: filesystem
  tool: read_file
  arguments:
    path: '/documents/template.md'

# Expose MCP tools to an LLM
- step: HUB_COMPLETION
  instruction: 'Read and summarize the template'
  builtinTools:
    - mcp: 'filesystem'
      only: ['read_file']
```

See the [AI Integration Guide](./guides/configuration/ai-integration.md#model-context-protocol-mcp-tools) for full documentation.

**🔄 Streaming LLM Responses**

OpenAI completion steps now support streaming mode. Tokens from the LLM are displayed in the Flow Monitor in real time as they arrive, providing immediate feedback for long completions.

- Streaming works with both plain text and JSON (`response_format: 'json_object'`) responses
- Tool call streaming is also supported
- Streaming is transparent — the final `flowContext.text` / `flowContext.object` output is identical to non-streaming mode
- The monitor stays open during streaming and fades only after the response is complete

**🏗️ Block Drop Integration (Swing)**

Trigger ProActions flows automatically when a block element is dropped in the Swing editor. This enables powerful automation for drag-and-drop scenarios such as cross-story copies, channel migrations, and version restores.

```yaml
- title: 'Auto-translate Dropped Content'
  hidden: true
  swing:
    onBlockDrop:
      enable: true
      runWhen:
        isCrossStory: true
      when: "options.sourceLanguage !== options.targetLanguage"
  flow:
    - step: DEEPL_TRANSLATE
      instruction: '{{ flowContext._dropInfo.xmlContent | safe }}'
      target_lang: '{{ flowContext._dropInfo.targetLanguage }}'
    - step: REPLACE_XML
      at: CURSOR_PARAGRAPH
```

Drop information is available via `flowContext._dropInfo` (blockType, xmlContent, textContent, sourceLanguage, targetLanguage, isCrossStory, etc.).

See the [Swing Integration Guide](./guides/configuration/swing-integration.md#block-drop-integration) for all configuration options.

**🧩 New Steps**

- **`PLATFORM`**: Route flow execution to platform-specific branches (`swing`, `prime`, `standalone`) with an optional `default` fallback. Enables writing portable flows that behave differently depending on where they run.

  ```yaml
  - step: PLATFORM
    swing:
      - step: REPLACE_TEXT
        at: CURSOR
    prime:
      - step: SHOW_RESPONSE
        title: 'Result'
    default:
      - step: SHOW_NOTIFICATION
        message: '{{ flowContext.text }}'
  ```

- **`GET_ZONES`**: Retrieve zone and linked document information from a DWP or Report. The result is stored in `flowContext.object` as an array of zone descriptors.

- **`GET_PURE_TEXT`**: Extract plain text from the document with XML tags removed and optional character normalization. Useful when you need clean, AI-consumable text without markup.

- **`TAG_PURE_TEXT`**: Mark segments of text in the document with a virtual or real XML tag. Useful for highlighting AI-identified segments (e.g., claims for fact-checking, passages for review).

- **`CLEAR_TAGS`**: Remove all virtual tags that were previously added by `TAG_PURE_TEXT`.

**⚙️ Step Improvements**

- **`SCRIPTING` — `returnAs` option**: Treat the script body as an expression and assign the result directly to a `flowContext` variable, without needing an explicit `return` statement. Useful for concise one-liner scripts.

  ```yaml
  - step: SCRIPTING
    script: flowContext.items.filter(i => i.active).length
    returnAs: activeCount
  ```

- **`SANITIZE` — `normalizeCharacters`**: New option to normalize typographic characters (curly quotes, em dashes, etc.) to their ASCII equivalents during sanitization.

- **`SHOW_RESPONSE` — Markdown mode**: Render the response as formatted Markdown in the modal dialog instead of plain text, enabling richer formatted output.

- **`FORM` step**: The `FORM` step now has full schema support, documentation, and examples. All field types and form configuration options are validated.

**📡 Monitor Interaction Steps**

Eight dedicated flow steps let you interact with and display data in the Flow Monitor directly from YAML — no scripting required:

_Interactive steps_ — pause the workflow and wait for user input:

- **`MONITOR_STATUS`**: Display a status message in the monitor during execution. Supports message types (`info`, `status`, `warning`, `success`, `error`) and an optional `clear` flag to reset previous messages.

  ```yaml
  - step: MONITOR_STATUS
    message: 'Connecting to API...'
    type: status

  - step: OPENAI_COMPLETION
    # ...

  - step: MONITOR_STATUS
    message: 'Done!'
    type: success
  ```

- **`MONITOR_CONFIRM`**: Ask the user a Yes/No question. The result is written to `flowContext.confirmed` (boolean) and can be named via `outputs`.

  ```yaml
  - step: MONITOR_CONFIRM
    message: 'Publish this article?'
    confirm: 'Yes, Publish'
    cancel: 'Not yet'
    confirmType: success

  - step: IF
    condition: '{{ flowContext.confirmed }}'
    then:
      - step: SHOW_NOTIFICATION
        message: 'Published!'
  ```

- **`MONITOR_CHOICE`**: Present a list of labelled options. Supports single and multi-select modes, optional custom text input alongside the choices, and a configurable default.

  ```yaml
  - step: MONITOR_CHOICE
    message: 'Select export format:'
    choices:
      - id: pdf
        label: 'PDF Document'
      - id: docx
        label: 'Word Document'
    outputs:
      - type: text
        name: format
  ```

- **`MONITOR_INPUT`**: Collect a single line of text from the user. Supports `placeholder`, `default`, `required`, `maxLength`, and a custom `errorMessage`.

  ```yaml
  - step: MONITOR_INPUT
    message: 'Enter a title:'
    placeholder: 'Breaking News...'
    required: true
    outputs:
      - type: text
        name: userTitle
  ```

_Display steps_ — render structured data in the monitor panel (no user input required):

- **`MONITOR_JSON`**: Render an object or array as syntax-highlighted JSON. Optional `title` and `maxDepth`.
- **`MONITOR_TABLE`**: Render an array of objects as a table. Optional `title` and `maxRows`.
- **`MONITOR_KEYVALUE`**: Render an object as a labeled key-value list. Optional `title`.
- **`MONITOR_METRICS`**: Render numerical metrics with labels. Optional `title`.

  ```yaml
  - step: MONITOR_METRICS
    title: 'Content Analysis'
    data:
      Words: '{{ flowContext.wordCount }}'
      'Reading Time': '{{ flowContext.readingTime }} min'
      'Readability Score': '{{ flowContext.score }}/10'
  ```

See the [Flow Monitor Guide](./guides/configuration/flow-monitor.md) for the full reference on all eight steps.

**🏗️ Reusable Blocks**

A new `BLOCKS` section in `AI_KIT` (and in individual sections/actions) lets you define reusable constants, variables, flows, and scripts that are shared across your configuration without copy-pasting.

- **`constants`**: Static values (strings, numbers) referenced as `{{ global.constants.myValue }}`
- **`vars`**: Dynamic expressions evaluated at resolve-time
- **`flows`**: Reusable step sequences callable via `CALL_FLOW`
- **`scripts`**: Reusable script strings for the `SCRIPTING` step

Blocks are namespaced: `global.*`, `section.*`, `local.*`, and `imports.*` (for imported modules).

```yaml
AI_KIT:
  BLOCKS:
    constants:
      apiBasePath: /SysConfig/ProActions/lib
      defaultModel: gpt-4o

  SECTIONS:
    - section: Editorial
      blocks:
        flows:
          normalize:
            - step: SANITIZE
            - step: REPLACE_TEXT
              at: FULL
      actions:
        - title: Normalize Story
          flow:
            - step: CALL_FLOW
              ref: section.flows.normalize
```

Blocks also support **module imports**: include a YAML file with `imports` and reference its blocks under `imports.<alias>.*`.

```yaml
- section: Translations
  imports:
    - from: '{{ global.constants.apiBasePath }}/translate.module.yaml'
      as: translate
  actions:
    - title: Translate to French
      flow:
        - step: CALL_FLOW
          ref: imports.translate.flows.toFrench
```

See the [Configuration Guide](./setup/configuration.md#reusable-blocks-and-imports-recommended) for the full reference.

**🔗 CALL_FLOW and Scoped Flow References**

`CALL_TEMPLATE` now has a `CALL_FLOW` alias and a new `ref` option for calling flows defined in `BLOCKS` namespaces. The legacy `name`-based lookup against `TEMPLATES:` continues to work unchanged.

```yaml
# Legacy — still works
- step: CALL_TEMPLATE
  name: 'My Template Name'

# New — scoped reference from BLOCKS
- step: CALL_FLOW
  ref: section.flows.normalize

# Dynamic ref resolved at runtime
- step: CALL_FLOW
  ref: '{{ flowContext.selectedFlow }}'
```

The `ref` value is a dot-path into the resolver scope (`global.flows.*`, `section.flows.*`, `local.flows.*`, `imports.<alias>.flows.*`).

**🔧 Custom Template Filters**

ProActions now ships a set of built-in Nunjucks filters for working with objects and arrays directly inside `{{ }}` expressions:

- **`pluck(path)`** — extract a nested property from every item in an array: `{{ flowContext.users | pluck('profile.name') | join(', ') }}`
- **`find(attr, value)`** — return the first array item where `attr === value`
- **`getAttr(attr)`** — safely read a property from an object (own properties only, no prototype traversal)
- **`compact`** — remove falsy values from an array
- **`flatten`** — flatten one level of nested arrays
- **`sum(attr?)`** — sum array values (or a numeric attribute of each item)

These can be chained with all standard Nunjucks filters (`upper`, `trim`, `truncate`, `replace`, `join`, etc.).

```yaml
- step: SET
  # Extract names of active users and join them into a comma list
  activeNames: "{{ flowContext.users | find('active', true) | pluck('name') | join(', ') }}"
  # Compact removes nulls/undefineds before further processing
  cleanList: "{{ flowContext.rawList | compact | join(' | ') }}"
```

See the [Nunjucks Templates Guide](./guides/howto/use-nunjucks-templates.md#custom-resolver-filters) for the full filter reference.

**🔲 Prime Integration Improvements**

Significant improvements to the Prime adapter:

- `isPageContext` support: Flows can now detect whether they are running in a page context within Prime, enabling page-aware logic.
- Improved XML read/write operations and context detection across a broader set of Prime editor modes.
- Better compatibility with the Prime 8 command palette and resizable palette via `CHANGE_VIEW_SIZE`.

**⚠️ BREAKING CHANGE: Client adapter data APIs are now async**

To remove blocking synchronous HTTP requests (Prime), the following `client` methods now need to be executed asynchronously:

- `getBasefolder(storyId, type)`
- `getUserData(name)`
- `setUserData(name, data)`

If you use these methods in `SCRIPTING`, migrate to async mode and `await`:

```yaml
- step: SCRIPTING
  mode: async
  script: |
    const prompts = await client.getUserData('prompts');
    prompts.push({ id: 'new', text: 'example' });
    await client.setUserData('prompts', prompts);
    flowContext.baseFolder = await client.getBasefolder(client.getDocumentId(), 'image');
    return flowContext;
```


### 1.1.0 - October 24 2025

**🎨 Interactive Diff Component**

A powerful new form component that allows users to review and selectively accept or reject text changes:

- **Interactive Mode**: Click checkboxes and toggles to include/exclude specific changes
  - Additions: Include or exclude new content
  - Deletions: Apply or keep original text
  - Replacements: Choose between previous or new text
- **Toolbar Controls**:
- - Zoom in/out for better readability
  - Accept All / Reject All buttons for bulk operations
  - Customizable button symbols and positioning
- **Real-time Updates**: Result text recalculates as selections change

Example usage:

```yaml
- step: FORM
  form:
    diffField:
      type: 'diff'
      mode: 'words'
      prevText: '{{ flowContext.original }}'
      text: '{{ flowContext.improved }}'
      interactive: true
      showDeletions: true
      diffControls:
        position: 'top'
        zoom: true
        acceptRejectAll: true
```

Perfect for AI text improvement workflows, translation review, and grammar correction tools.

**⚡ Inline Steps Feature**

Improved user experience for UI steps with background processing:

- **FORM**, **USER_SELECT**, and **SHOW_RESPONSE** now support `inlineSteps`
- Modal displays immediately with loading indicator
- Background steps execute while modal is visible
- Results seamlessly populate the UI when processing completes
- Customizable loading messages via `loadingText` configuration
- Error handling displays errors in-place within the modal

Example usage:

```yaml
- step: USER_SELECT
  inlineSteps:
    - step: HUB_COMPLETION
      instruction: 'Generate 6 creative headlines for: {textContent}'
      response_format: 'list'
  promptText: 'Select a headline'
  enableKeyboardControl: true
```

This eliminates perceived wait time for long-running LLM calls and provides immediate user feedback.

**🔧 Swing Integration System**

Native integration with Swing Web Client UI components:

- **inlineMenu**: Add buttons to the inline editor toolbar
  - Appears when editing content in report, story, or DWP contexts and selecting text.
  - Supports conditional visibility and enablement via expressions
  - Configuration: `label`, `icon`, `allowReadOnly`, `isEnabled`, `isActive`

- **objectAction**: Add actions to the object three dots menu (e.g. in search results or preview)
  - Works across a lot of contexts

- **contextMenu**: Add items to right-click context menus
  - Appears in Explorer and other contexts
  - Same configuration options as objectAction

Example configuration:

```yaml
- title: 'Shorten Selection'
  swing:
    inlineMenu:
      enable: true
      icon: 'fa fa-compress'
      label: 'Shorten'
      allowReadOnly: false
      isEnabled: "{{ ctx.activeDocument.getSelection().getNode().elementName == 'p' }}"
  flow:
    - step: HUB_COMPLETION
      instruction: 'Shorten this text: {selectedText}'
    - step: REPLACE_TEXT
      at: CURSOR
```

**⚠️ BREAKING CHANGE: New Deployment Structure**

ProActions now uses a **single unified bundle** for deployment. The old multi-file structure is no longer supported:

**Old Structure (v1.0.x):**

```
~/methode-servlets/conf/swing/com.eidosmedia.swing.web-app/app/plugins/
├── pro-actions-core/
│   ├── pro-actions-bundle.js
│   ├── pro-actions-command.js
│   └── pro-actions-config.js
└── pro-actions-ext-ai-kit/
    └── proactions-ai-kit.js
```

**New Structure (v1.1.0+):**

```
~/methode-servlets/conf/swing/com.eidosmedia.swing.web-app/app/plugins/
└── pro-actions/
    ├── pro-actions-{version}.js    (single bundle file)
    └── pro-actions-config.js       (configuration file)
```

**Migration Steps:**

1. **Backup configuration**:
   - Save a copy of `plugins/pro-actions-core/pro-actions-config.js`

2. **Remove old files**:
   - Delete `plugins/pro-actions-core/` directory
   - Delete `plugins/pro-actions-ext-ai-kit/` directory

3. **Install new bundle**:
   - Create `plugins/pro-actions/` directory
   - Copy `pro-actions-{version}.js` from the release package
   - Copy your backed-up `pro-actions-config.js` to `plugins/pro-actions/`

4. **Verify configuration**:
   - Review `pro-actions-config.js` for any path references (usually none needed)
   - Restart your Swing client to load the new bundle

**Benefits:**

- Simplified deployment with single bundle file
- Reduced complexity in installation process
- Easier version management and updates
- Consistent file structure across installations

**⚠️ TEXT_MENU Deprecation**

The TEXT_MENU component is now **deprecated** in favor of Swing's native `inlineMenu` integration:

- Using TEXT_MENU in recent Swing versions will show both the inline toolbar and TEXT_MENU
- The two components are not compatible and create a poor user experience
- **Action Required**: Migrate your TEXT_MENU configurations to use `swing.inlineMenu`
- TEXT_MENU will be removed in a future version

Migration example:

```yaml
# OLD - Deprecated
TEXT_MENU:
  story:
    items:
      - type: 'button'
        icon: 'fa fa-icon'
        title: 'My Action'
        flowRef: 'My Action'
    # ...

# NEW - Recommended
title: 'My Action'
swing:
  inlineMenu:
    enable: true
    icon: 'fa fa-icon'
    label: 'My Action'
```

**🤖 Enhanced Chat Completion Steps**

Significant improvements to all chat completion steps (OPENAI_COMPLETION, HUB_COMPLETION, etc.):

- **Tool/Function Calling**:
  - Define custom functions that the LLM can invoke during conversation
  - Functions can execute ProActions workflow steps, scripts, or templates
  - Support for required and optional parameters
  - Multi-turn conversations with tool results

  ```yaml
  - step: OPENAI_COMPLETION
    instruction: 'Ask the user a question using the tool askUser'
    functions:
      - name: askUser
        description: 'Ask the user a question'
        required_params: ['question']
        steps:
          - step: USER_PROMPT
            promptText: '{{ flowContext.question }}'
  ```

- **Binary Input/Output Support**:
  - Audio input support for speech-capable models
  - Audio output support with automatic playback
  - Image input improvements with better multimodal handling
  - Base64 and blob handling for all media types

- **Structured Outputs**:
  - Enhanced `response_format` with `json_object` option.

- **Configuration Improvements**:
  - Support for conversation history via `messages` array

**📚 Prompt Library Integration**

Reuse and centrally manage prompts:

- Reference stored prompts using `promptId` configuration
- Prompts managed in Swing Prompt Management UI
- Supports both system and user prompts
- Variable resolution works within stored prompts
- Reduces duplication and improves governance

Example:

```yaml
- step: OPENAI_COMPLETION
  promptId: 'd112a306-8f7a-43bc-8f38-94161b6a91cd'
  # All other configurations remain available
```

**🔄 New Control Flow Steps**

Advanced workflow control capabilities:

- **BREAK**: Exit loops conditionally

  ```yaml
  - step: WHILE
    condition: '{{ flowContext.counter < 10 }}'
    do:
      - step: SET
        counter: '{{ flowContext.counter + 1 }}'
      - step: BREAK
        condition: '{{ flowContext.counter == 5 }}'
  ```

- **FOR**: Numeric and array iteration

  ```yaml
  # Numeric iteration
  - step: FOR
    var: 'i'
    start: 1
    end: 5
    do:
      - step: SET
        item: '{{ flowContext.i }}'

  # Array iteration
  - step: FOR
    var: 'item'
    items: '{{ flowContext.myList }}'
    do:
      - step: SET
        current: '{{ flowContext.item }}'
  ```

- **TRY**: Error handling with catch blocks

  ```yaml
  - step: TRY
    try:
      - step: SET
        value: '{{ riskyOperation }}'
    catch:
      - step: SET
        error: '{{ flowContext.error }}'
        fallbackValue: 'default'
  ```

- **PARALLEL**: Execute multiple step sequences concurrently

  ```yaml
  - step: PARALLEL
    steps:
      - # Branch 1
        - step: HUB_COMPLETION
          instruction: 'Generate headline'
      - # Branch 2
        - step: HUB_COMPLETION
          instruction: 'Generate summary'
  ```

**🎵 New Steps**

- **PLAY_AUDIO**: Display a customizable floating audio player
  - Supports data URLs, HTTP URLs, and base64 audio
  - Configurable theme (dark/light or custom colors)
  - Position control (9 anchor positions)
  - Progress bar, volume controls, and time display
  - Autoplay support with browser compatibility fallbacks
  - Draggable player with Shadow DOM isolation

  ```yaml
  - step: PLAY_AUDIO
    in: '{{ flowContext.audioDataUrl }}'
    autoplay: true
    theme: 'dark'
    position: 'bottom-right'
    showProgress: true
    showVolume: true
  ```

- **BASE64_TO_BLOB**: Convert base64 strings to blobs
  - Essential for handling binary data in workflows
  - Supports custom MIME types
  - Integrates with other steps requiring blob inputs

**📝 FORM Enhancements**

- **formConfig**: New configuration object for FORM step behavior
  - Control modal sizing and appearance (width, height, dialogSize)
  - Fullscreen mode support
  - Typography customization (fonts, line heights)

  ```yaml
  - step: FORM
    title: 'Large Review Form'
    formConfig:
      dialogSize: 'xl'
      height: '80vh'
      diffFontSize: '14px'
    form:
      reviewField:
        type: 'diff'
        mode: 'words'
        prevText: '{{ flowContext.original }}'
        text: '{{ flowContext.improved }}'
  ```

  See [Forms and Inputs Guide](./guides/configuration/forms-and-inputs.md#modal-configuration) for complete formConfig options.

- **Textarea Resize**: Enhanced textarea component
  - `rows` configuration for initial height
  - `resize` option: `none`, `vertical`, `horizontal`, `both`
  - Better control over form layout

  ```yaml
  - step: FORM
    form:
      notes:
        type: 'textarea'
        rows: 10
        resize: 'vertical'
  ```

**🎯 USER_SELECT Keyboard Control**

- **enableKeyboardControl**: Navigate selection lists with keyboard
  - Arrow keys for navigation
  - Enter to select
  - Escape to cancel
  - Improves accessibility and power-user workflows

  ```yaml
  - step: USER_SELECT
    enableKeyboardControl: true
    promptText: 'Choose an option'
  ```

**🔐 Admin Actions**

- **admin** flag: Mark actions as administrative
  - Hidden from regular command palette
  - Requires typing the full action name to access
  - Useful for configuration reload, cache clearing, debugging tools
  - Prevents accidental execution of sensitive operations

  ```yaml
  - title: 'Reload Configuration'
    admin: true
    flow:
      - step: # Reload steps
  ```

**✨ Conditional Visibility with Templates**

- **isAvailable** now supports Nunjucks template expressions
  - Use `{{ }}` syntax for dynamic conditions

  ```yaml
  - title: 'Report Action'
    isAvailable: "{{ ctx.activeObject.getInfo().type == 'EOM::CompoundStory' }}"
  ```

**Additional Improvements**

- JSON schema updates for all new features
- New document lifecycle operations available in workflows and scripts:
  - `SAVE_DOCUMENT` step supports optional `close` behavior
  - `CLOSE_DOCUMENT` step closes the current document
  - Scripting API now supports `client.saveDocument({ close: true })` and `client.closeDocument()`

### 1.0.12 - February 7, 2025

**🎯 Slash Menu**

The new slash menu brings Notion-style command access to ProActions:

- Type `/` anywhere in the editor to open the slash menu
- Configuration consistent with TEXT_MENU and ONECLICK_MENU
- Provides quick access to actions without keyboard shortcuts

**New Steps**

- **END_IF** - Conditionally terminate flow execution
- **SANITIZE** - Validate and repair XML content
  - Essential for AI-generated XML where validity is uncertain
  - Can automatically fix common XML issues
- **ELEVENLABS_TTS** - ElevenLabs Text-to-Speech integration
- **ELEVENLABS_STT** - ElevenLabs Speech-to-Text integration

**Enhanced Features**

- **HUB_YOUTUBE_UPLOAD**
  - Now supports uploading video thumbnails
  - Improved video upload workflow

- **Méthode XML to Markdown**
  - New `client.xmlToMarkdown(xml)` function
  - Simplifies content export and preview

**Prime 8 Integration**

- Full integration with Prime 8 Command Palette framework
- Specialized visualization for Prime 8
- **CHANGE_VIEW_SIZE** step to control palette sizing

**ProActions Financial Services**

New client functions for financial services integration:

- `client.getContainerTextContent()` - Fetch full report text in Swing and Prime
- `client.getContainerXmlContents()` - Retrieve XML and LOIDs for all report sections
- Enhanced `client.getTextContent()` - Returns selected section text
- Enhanced `client.getXmlContent()` - Returns selected section XML

**Improvements**

- `client.isReadonly()` now checks if active user holds the lock on the active object

---

### 1.0.11 - April 28, 2025

**🎓 Schema Support**

Created and registered ProActions AI-Kit schema for enhanced development:

- Auto-completion for configuration files
- Real-time validation
- Inline documentation
- Registered at [schemastore.org](https://schemastore.org)
- Reduces configuration errors significantly

**Configuration Enhancements**

_Actions and Sections:_

- **`app` attribute** - Specify if action/section is for "swing" or "prime"
- **`category` attribute** - Set sub-category for better organization in Prime Panel
- **`hiddenInPanel` attribute** - Hide specific actions in Prime Panel
- **`sectionIcon` attribute** - Custom icon for sections in Prime Panel
- **`panelContext` attribute** - Specify context actions for selected elements

**YouTube Integration**

New steps for YouTube video management:

- **HUB_YOUTUBE_AUTH_INIT** - Initialize OAuth 2.0 authentication
- **HUB_YOUTUBE_AUTH_STATUS** - Check authentication status
- **HUB_YOUTUBE_AUTH_LOGOUT** - Logout authenticated accounts
- **HUB_YOUTUBE_UPLOAD** - Upload videos to YouTube

**User Data Persistence**

New client functions for user profile data:

- `getUserData()` - Retrieve stored user data
- `setUserData()` - Save user-specific data for future sessions

**Development Support**

- **Training Mode** - Load configs from local development server
- Enable with `trainingMode` configuration
- Use with [proactions-dev](https://github.com/em-al-wi/proactions-dev)

**Prime Enhancements**

- Improved XML management with `getXmlAtXpath` and `replaceXmlAtXpath`
- Better XML insert location support
- Improved read-only state detection using `IsModifyable()`

**Step Improvements**

- **CLIPBOARD** - Browser compatibility fallbacks
- **FORM**
  - Key-value pairs for select options and radio items
  - New form components: datetime, color, range
- **REST** - `formData` attribute for multipart form data

**Branding**

- Product name changed to "Eidosmedia ProActions" (formerly "Swing ProActions")

---

### 1.0.10 - February 27, 2025

**📝 Enhanced Forms**

New FormComponents for richer user interfaces:

- **headline** - Section headers in forms
- **html** - Rich HTML content display
- **markdown** - Markdown rendering in forms
- **diff** - Side-by-side text comparison
- **hr** - Horizontal rules for visual separation
- **Custom buttons** - Define multiple action buttons

**New Steps**

- **MARKDOWN_TO_HTML** - Transform Markdown to HTML format

**Enhanced Steps**

- **USER_SELECT**
  - `infoTitle` attribute for additional context
  - `infoText` attribute for descriptive information above list

- **SCRIPTING**
  - Now supports asynchronous JavaScript execution
  - Greater workflow flexibility

- **SHOW_RESPONSE**
  - HTML response rendering support

**Language Support**

- ✨ **Spanish Translation** - ProActions now available in Spanish
- Supported languages: English, Spanish, French, Italian, German

**Context Handling**

- Multiple contexts support in sections and actions
- Improved fallback mechanisms for Swing 5.2022.10
- Context support added for Prime
- Better context detection across platforms

**Editor Protection**

- Steps that modify content now check for read-only mode:
  - INSERT_TEXT, INSERT_XML
  - REPLACE_TEXT, REPLACE_XML
  - INSERT_LIST

**Text and Oneclick Menu**

- `forElement` configuration now supports "DummyText"

**Maintenance**

- Updated all dependencies to latest versions

---

### 1.0.9 - January 26, 2025

**DeepL Write Integration**

- **New Step: DEEPL_WRITE** - Send text for improvement to DeepL
- Enables text refinement and optimization
- Note: DeepL Write API does not support XML format text

**INSERT_XML and INSERT_LIST Enhancements**

New **position** parameter for flexible insertion:

- `insertInline` (default) - Insert at cursor position
- `insertBefore` - Insert before cursor context
- `insertAfter` - Insert after cursor context
- Effective in Swing when `at` is set to `"CURSOR"`

---

### 1.0.8 - December 13, 2024

**📊 Progress Bar Element**

Visual feedback for long-running workflows:

- **SHOW_PROGRESS** - Display progress bar
- **UPDATE_PROGRESS** - Update progress state
- **HIDE_PROGRESS** - Hide progress bar
- Auto-hides on workflow completion or error

**Dashboard Actions**

- AI-Kit now supports dashboard actions
- New **context** configuration for actions and sections
- Ensures actions only visible in relevant contexts
- ⚠️ **Action Required**: Add `context:` config for context-specific actions

**Translation Management**

- Translations now built into solution
- ⚠️ **Action Required**: Remove standalone locale files

**One-Click Menu Enhancement**

- New `forTags` configuration option
- Define menu appearance based on parent tags

**Content Filtering**

- Optional parameters for `client.getTextContent` and `client.getXmlContent`
- `GetTextContentOptions` - Filter returned content
- `GetXmlContentOptions` - Filter returned XML
- Example: Exclude specific tags from content

**Minor Updates**

- **Hide sections** - Hide all actions within a section
- **Azure OpenAI** - Updated IMAGE_GENERATION API version to `2024-10-01-preview`
- **client.getContext()** - New function to identify user's current context

---

### 1.0.7 - November 14, 2024

**🎨 Context-Based Menus**

Two new highly configurable menus:

_Text Selection Menu:_

- Appears when text is selected
- Quick access to text-specific actions
- Fully customizable per context
- Permission-based visibility

_One-Click Menu:_

- Appears on left side of content items
- Activated when cursor is placed inside content
- Context-sensitive actions
- Per-action permissions

**External Configuration Management**

- **!include directive** - Include external SysConfig files
- Modular configuration management
- Better organization and maintainability
- Example: `!include /SysConfig/ProActions/services.yaml`

**New Steps**

- **CLEAR_SELECTION** - Clear text selection without replacement
- Useful before INSERT_TEXT or INSERT_XML steps

**Stability AI Enhancement**

- **SEARCH_AND_RECOLOR** - Identify and recolor image components
- Uses Stability AI's recoloring model

**LLM Response Format**

- **response_format** option for OpenAI ChatGPT and Google Gemini
- Enforce JSON response structure
- Custom schema support
- Alias `list` for JSON list enforcement

**Step Improvements**

- **DOWNLOAD** - Now supports downloading blobs
- **INSERT_LIST / TO_LIST** - Improved list parsing logic
  - Removes double newlines
  - Cleaner list generation

---

### 1.0.6 - October 11, 2024

**🎤 Speech Services**

- New Text-to-Speech (TTS) models from OpenAI and Azure OpenAI
- High-quality voice synthesis
- Multiple voice options

**Document Metadata**

- New placeholders for document name and issue date
- Enhanced client functions for metadata access

**🔒 Permissions System**

Configure access control for sections and actions:

- `forUsers` - User-level permissions
- `forGroups` - Group-level permissions
- `forTeams` - Team-level permissions

**New Steps**

- **DEBUG** - Write flowContext to developer console
- Creates breakpoint for debugging
- Essential for workflow troubleshooting

**Templating Engine**

- Integrated Mozilla's Nunjucks templating engine
- Enhanced string handling
- Powerful placeholder system

**Step Enhancements**

- **IF, WHILE, SWITCH** - New `condition` attribute
  - Supports string placeholders with Nunjucks
  - More flexible conditional logic

- **SET, SWITCH** - Simplified syntax
  - Replaced name/expression attributes with key-value pairs
  - Cleaner configuration

**User Experience**

- **Instant Actions** - Execute on ProActions icon click
- Improved workflow initiation

**Technical Updates**

- Removed deprecated DOMSubtreeModified MutationEvent
- Better cursor location monitoring

---

### 1.0.5 - September 3, 2024

**New Steps**

- **DOWNLOAD_TEXT** - Download text content from flow
- **WRITE_CLIPBOARD** - Write to user clipboard

**FILE_UPLOAD Enhancement**

- New output types: `text` and `arrayBuffer`
- More flexible file handling

**Dynamic Placeholders**

- `reportId` - Report identifier
- `reportLoid` - Report LOID
- `reportUuid` - Report UUID

**API Enhancements**

- Access Metadata, System Metadata, Usage Tickets, Virtual Attributes
- Support for both story and report (container) contexts

---

### 1.0.4 - August 15, 2024

**Performance Improvements**

- Faster placeholder resolution
- Asynchronous configuration loading
- Improved overall responsiveness

---

### 1.0.3 - August 14, 2024

**🎨 Stability AI Integration**

New steps for image processing:

- **STABILITY_AI_UPSCALE** - Enhance image resolution
- **STABILITY_AI_OUTPAINT** - Extend image boundaries
- **STABILITY_AI_SEARCH_AND_REPLACE** - Find and replace in images

**Placeholder Improvements**

- Option to omit text when variable cannot be resolved
- More graceful handling of missing variables

**Flow Control**

- Continue flow even when `then` or `else` block is missing
- More resilient conditional execution

**INSERT_LIST Enhancement**

- Add entries to existing lists
- More flexible list management

**REST Step Enhancement**

- Store response object in variable
- `requestOptions` configuration
- More control over REST calls

**API Functions**

- `getSiteConfigString()` - Read from siteConfiguration
- Better integration with Méthode configuration

**New Placeholders**

- `uuid` - Document UUID
- `loid` - Document LOID
- `appName` - Application name (swing/prime)

---

## Migration Notes

### Upgrading to 1.1.0

**⚠️ TEXT_MENU Deprecation (Action Required)**

- Migrate all TEXT_MENU configurations to use `swing.inlineMenu`
- Using TEXT_MENU in recent Swing versions causes UI conflicts
- TEXT_MENU will be removed in a future version

Migration steps:

1. Identify all actions using TEXT_MENU configuration
2. Replace with `swing.inlineMenu` configuration
3. Test in your Swing environment
4. Remove TEXT_MENU sections from configuration

**New Features**

- All new features are opt-in and backward compatible
- Interactive diff available in FORM step with `type: 'diff'`
- Inline steps can be added to existing FORM, USER_SELECT, and SHOW_RESPONSE steps
- Swing integrations work alongside existing action definitions
- New control flow steps (BREAK, FOR, TRY, PARALLEL) available immediately

**Recommended Actions**

- Consider using `inlineSteps` for actions with LLM calls to improve UX
- Explore interactive diff for text improvement workflows
- Review admin actions and add `admin: true` flag where appropriate
- Update `isAvailable` conditions to use Nunjucks templates for better readability

### Upgrading to 1.0.12

- No breaking changes
- New features are opt-in
- Consider enabling Slash Menu for improved UX

### Upgrading to 1.0.11

- Schema support requires VS Code YAML extension update

### Upgrading to 1.0.8

- Remove standalone ProActions locale files (now built-in)
- Add `context:` configuration to context-specific actions

---

## Getting Help

- **Documentation**: See [Getting Started Guide](./getting-started/your-first-flow.md)
- **Issues**: Check existing issues or report new ones
- **Community**: Join ProActions Developer Community
- **Support**: Contact Eidosmedia support team
