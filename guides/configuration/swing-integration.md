# Swing Integration

This guide covers integrating ProActions into the **Swing Web Client**, including embedding actions in toolbars, object panels, image toolbars, and using the ProActions API for programmatic access.

## Overview

ProActions can be integrated into Swing through two main approaches:

1. **UI Integration** - Embed ProActions in toolbars, object panels, and component toolbars
2. **API Integration** - Programmatic access to ProActions functionality via JavaScript API

Both approaches allow you to seamlessly integrate AI-powered workflows into your Swing workflows.

## Prerequisites

- **ProActions Installed**: ProActions extension properly installed on Swing
- **Configuration File**: `/SysConfig/pro-actions.ai-kit.yaml` configured with actions
- **Swing Extensions Knowledge**: Basic understanding of Swing extension development
- **JavaScript Knowledge**: Familiarity with JavaScript for custom integrations

## Event-Driven Action Triggers

Actions can subscribe to runtime events and execute automatically without menu interaction.

### Action-Level Event Subscriptions

Configure subscriptions with the `events` property on an action:

```yaml
- id: auto-postprocess
  title: "Auto post-process"
  hidden: true
  events:
    - source: "aikit"
      name: "aiKitFlowCompleted"
      condition: "{{ flowContext._eventPayload.actionId == 'source-action' }}"
      ifRunning: "queue"

  flow:
    - step: SHOW_NOTIFICATION
      message: "Post-processing triggered"
```

### Supported Event Sources

- `aikit` - AI-Kit lifecycle events (`aiKitReady`, `aiKitFlowCompleted`, `aiKitFlowCancelled`, `aiKitFlowError`)
- `system` - System lifecycle events (`onEditorReady`)
- `custom` - User-defined events from `EMIT_EVENT` / `DISPATCH_EVENT` steps

### AI-Kit Lifecycle Event Reference

Use these lifecycle events with `source: "aikit"` in action subscriptions:

| Event | Emitted when | Common condition/payload usage |
|-------|--------------|-------------------------------|
| `aiKitReady` | AI-Kit has loaded configuration and registered actions. | Usually no payload filter needed. Use for startup automation. |
| `aiKitFlowCompleted` | Any action flow completes successfully. | Filter by source action id: `{{ flowContext._eventPayload.actionId == 'my-action-id' }}` |

Example:

```yaml
- id: startup-check
  title: "Startup Check"
  hidden: true
  events:
    - source: "aikit"
      name: "aiKitReady"
  flow:
    - step: SHOW_NOTIFICATION
      message: "AI-Kit is ready"

- id: postprocess-after-summary
  title: "Postprocess After Summary"
  hidden: true
  events:
    - source: "aikit"
      name: "aiKitFlowCompleted"
      condition: "{{ flowContext._eventPayload.actionId == 'summary-action' }}"
  flow:
    - step: SHOW_NOTIFICATION
      message: "Summary flow completed"
```

### Concurrency (`ifRunning`)

| Value | Behavior |
|-------|----------|
| `skip` | Ignore new trigger while action is running |
| `queue` | Queue trigger and run after current execution completes |
| `parallel` | Start additional execution immediately |

`condition` is optional and evaluated per event. If evaluation fails, the trigger is skipped.

### Access Event Data In Flows

Triggered actions receive event metadata in `flowContext`:

- `flowContext._event` - Complete event object
- `flowContext._eventName` - Event name
- `flowContext._eventPayload` - Event payload

Use these variables in subsequent steps:

```yaml
- id: auto-postprocess
  title: "Auto post-process"
  hidden: true
  events:
    - source: "aikit"
      name: "aiKitFlowCompleted"
      condition: "{{ flowContext._eventPayload.actionId == 'source-action' }}"

  flow:
    - step: SHOW_NOTIFICATION
      message: "Triggered by {{ flowContext._eventPayload.actionId }}"
```

`events.condition` uses resolver boolean evaluation. Use `flowContext._event`, `flowContext._eventName`, and `flowContext._eventPayload` inside the condition.

## Native Swing Integrations (v1.1.0+)

Starting with v1.1.0, ProActions supports **declarative integration** with Swing's native UI components directly in your action configuration. This eliminates the need for custom JavaScript code for common integration scenarios.

### Overview

Four integration types are available:

- **inlineMenu** - Add buttons to the inline text editor toolbar
- **objectAction** - Add actions to object three-dots menu
- **contextMenu** - Add items to right-click context menus
- **onBlockDrop** - Trigger actions when blocks are dropped in the editor

All four are configured using the `swing:` property in your action definition.

### Inline Toolbar Integration

Add buttons to the inline text editor toolbar that appears when selecting text in the editor.

#### Configuration

```yaml
- title: "Shorten Selection"
  swing:
    inlineMenu:
      enable: true
      label: "Shorten"
      icon: "fa fa-compress"
      allowReadOnly: false
      isEnabled: "{{ client.getMetadata().metadata.story.type == 'WireStory' }}"
      isActive: "{{ true }}"
      showIn:
        inlinetoolbar: true
        slashmenu: true
        blockmenu: true
  flow:
    - step: HUB_COMPLETION
      instruction: "Shorten: {selectedText}"
    - step: REPLACE_TEXT
      at: CURSOR
```

#### Configuration Options

| Property | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `enable` | boolean | Yes | - | Enable this integration |
| `label` | string | No | Action title | Button label text |
| `icon` | string | No | - | FontAwesome icon class (e.g., `"fa fa-magic"`) |
| `allowReadOnly` | boolean | No | `false` | Allow button to be active in read-only mode |
| `isEnabled` | string | No | `"{{ true }}"` | Nunjucks expression for button enablement |
| `isActive` | string | No | `"{{ true }}"` | Nunjucks expression for button visibility |
| `showIn` | object | No | all `true` | Source-level visibility flags evaluated before `isActive` |

`showIn` supports:

- `inlinetoolbar` - visibility for inline text-selection toolbar
- `slashmenu` - visibility for contextual slash menu (`ctx.component.getType() == "contextualtoolbar"` and `getSubType() == "slashmenu"`)
- `blockmenu` - visibility for contextual block menu (`ctx.component.getType() == "contextualtoolbar"` and `getSubType() == "blockmenu"`)

#### Visibility Evaluation Order

Swing visibility for `inlineMenu` is resolved inside command `isActive` in this order:

1. `showIn` source gate
2. integration-level `swing.inlineMenu.isActive`
3. action-level `action.isAvailable`

`isEnabled` is evaluated separately and controls enabled/disabled state only.

#### Where It Appears

The inline integration can appear from these command sources:
- **Inline text selection** - `ctx.component.getType() === "inlinetoolbar"`
- **Contextual block menu** - `ctx.component.getType() === "contextualtoolbar"` and `ctx.component.getSubType() === "blockmenu"`
- **Contextual slash menu** - `ctx.component.getType() === "contextualtoolbar"` and `ctx.component.getSubType() === "slashmenu"`

Use `showIn` to restrict where the action is shown.

#### Conditional Visibility Examples

Use Nunjucks expressions to control when buttons are enabled or visible:

**Only enable for paragraph elements:**

```yaml
swing:
  inlineMenu:
    enable: true
    label: "Format"
    isEnabled: "{{ ctx.activeDocument.getSelection().getNode().elementName == 'p' }}"
```

**Combine multiple conditions:**

```yaml
swing:
  inlineMenu:
    enable: true
    label: "Improve Paragraph"
    isEnabled: "{{ ctx.activeDocument.getSelection().getNode().elementName == 'p' and client.getChannel() == 'mychannel' }}"
```

**Check content item type:**

```yaml
swing:
  inlineMenu:
    enable: true
    label: "Headline Tool"
    isEnabled: "{{ ctx.activeDocument.getSelection().getNode().elementName == 'p' }}"
```

#### Complete Example

```yaml
- title: "Improve Selected Text"
  description: "Use AI to improve the selected text"
  swing:
    inlineMenu:
      enable: true
      label: "Improve"
      icon: "fa fa-magic"
      allowReadOnly: false
      isEnabled: "{{ ctx.activeDocument.getSelection().getNode().elementName == 'p' }}"
  flow:
    - step: HUB_COMPLETION
      behavior: "You are a writing assistant that improves text clarity and style."
      instruction: "Improve this text: {selectedText}"

    - step: REPLACE_TEXT
      at: CURSOR
```

### Object Action Integration

Add actions to the object three-dots menu that appears in search results, preview panes, and explorer grids.

#### Configuration

```yaml
- title: "Generate Summary"
  swing:
    objectAction:
      enable: true
  flow:
    - step: HUB_COMPLETION
      instruction: "Create a brief summary of: {textContent}"

    - step: SHOW_NOTIFICATION
      message: "Summary: {text}"
```

#### Configuration Options

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `enable` | boolean | Yes | Enable this integration |

#### Where It Appears

Object actions appear in:
- **Search results** - Three-dots menu for each result
- **Preview pane** - Object actions menu
- **Explorer grid** - Context actions for objects

#### Notes

- Object actions use the action's `title` property for the menu label
- Custom labels and icons are not supported in the object menu
- The action executes in the context of the selected object

#### Example

```yaml
- title: "Extract Keywords"
  description: "Extract keywords from the document"
  swing:
    objectAction:
      enable: true
  flow:
    - step: HUB_COMPLETION
      instruction: "Extract 10 relevant keywords from: {textContent}"
      response_format: "list"

    # ...
```

### Context Menu Integration

Add items to right-click context menus throughout Swing.

#### Configuration

```yaml
- title: "Process with AI"
  swing:
    contextMenu:
      enable: true
  flow:
    - step: HUB_COMPLETION
      instruction: "Analyze: {textContent}"

    # ...
```

#### Configuration Options

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `enable` | boolean | Yes | Enable this integration |

#### Where It Appears

Context menu items appear in:
- **Explorer** - Right-click menu on objects
- **Various contexts** - Context-specific right-click menus

#### Notes

- Context menu items use the action's `title` property for the menu label
- Custom labels and icons are not supported
- The action executes in the context of the right-clicked item

#### Complete Example

```yaml
- title: "Quick Translate"
  description: "Translate content to French"
  swing:
    contextMenu:
      enable: true
  flow:
    - step: DEEPL_TRANSLATE
      instruction: "{textContent}"
      target_lang: "FR"

    - step: SHOW_RESPONSE
      title: "Translation"
```

### Block Drop Integration

Trigger actions automatically when a block element is dropped in the editor. This is useful for processing content during drag-and-drop operations between stories, from old versions, or across channels.

#### Configuration

```yaml
- title: "Auto-translate Dropped Content"
  swing:
    onBlockDrop:
      enable: true
      runWhen:
        isCrossStory: true
      when: "options.sourceLanguage !== options.targetLanguage"
  flow:
    - step: DEEPL_TRANSLATE
      instruction: "{{ flowContext._dropInfo.xmlContent | safe }}"
      target_lang: "{{ flowContext._dropInfo.targetLanguage }}"
    - step: REPLACE_XML
      at: CURSOR_PARAGRAPH
```

#### Configuration Options

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `enable` | boolean | Yes | Enable this integration |
| `runWhen` | object | No | Simple conditions that must all match for the action to run |
| `when` | string | No | JavaScript expression for complex conditions (evaluated after `runWhen`) |

#### runWhen Conditions

Simple conditions based on block drop options. All specified conditions must match:

| Condition | Type | Description |
|-----------|------|-------------|
| `isCrossStory` | boolean | Block is being moved between different stories |
| `fromOldVersion` | boolean | Content is being restored from a previous version |
| `betweenChannelCopy` | boolean | Block is being copied between different channels |
| `isSameContainer` | boolean | Block is moved within the same container |
| `blockType` | string | Block type (e.g., `"image"`, `"video"`) |

**Condition matching:**
- `true` - The option must be truthy
- `false` - The option must be falsy
- String value - Requires exact match

#### Custom Expressions (when)

Use JavaScript expressions for complex conditions. The expression has access to:

- `options` - Block drop information object
- `ctx` - Swing editor context

```yaml
swing:
  onBlockDrop:
    enable: true
    when: "options.sourceLanguage !== options.targetLanguage && options.textContent.length > 100"
```

#### Accessing Block Drop Data in Flow

When an action is triggered by a block drop, the drop information is available in `flowContext._dropInfo`:

| Property | Type | Description |
|----------|------|-------------|
| `blockType` | string | Type of the dropped block (e.g., `"image"`, `"video"`) |
| `textContent` | string | Plain text content of the dropped block |
| `xmlContent` | string | XML representation of the dropped block |
| `htmlContent` | string | HTML representation of the dropped block |
| `isCrossStory` | boolean | Whether block is from a different story |
| `fromOldVersion` | boolean | Whether content is from a previous version |
| `sourceStoryId` | string | ID of the source story |
| `targetStoryId` | string | ID of the target story |
| `sourceLanguage` | string | Language of the source story |
| `targetLanguage` | string | Language of the target story |
| `betweenChannelCopy` | boolean | Whether copying between channels |
| `isSameContainer` | boolean | Whether moving within the same container |
| `referenceObject` | object | Reference to associated object (path, srcPreview) |
| `txtPath` | string | XPath to the dropped block element |

**Convenience aliases** are also available at the flowContext root level:
- `flowContext._droppedContent` - First available of: textContent, xmlContent, or htmlContent
- `flowContext._droppedBlockType` - The block type

#### Example: Access Drop Data

```yaml
flow:
  - step: IF
    test: "{{ flowContext._dropInfo.isCrossStory }}"
    then:
      - step: DEEPL_TRANSLATE
        instruction: "{{ flowContext._dropInfo.textContent }}"
        target_lang: "{{ flowContext._dropInfo.targetLanguage }}"
      - step: SHOW_NOTIFICATION
        message: "Translated from {{ flowContext._dropInfo.sourceLanguage }}"
```

#### Where It Works

Block drop integration works in:
- **Story editor** - When dropping blocks into stories
- **Report editor** - When dropping blocks into reports

#### Example: Translate Cross-Story Drops

Automatically translate content when dropping from a story in a different language:

```yaml
- title: "Auto-translate Dropped Content"
  hidden: true
  monitor:
    enabled: false
  swing:
    onBlockDrop:
      enable: true
      runWhen:
        isCrossStory: true
      when: "options.sourceLanguage !== options.targetLanguage"
  flow:
    - step: DEEPL_TRANSLATE
      instruction: "{{ flowContext._dropInfo.xmlContent | safe }}"
      target_lang: "{{ flowContext._dropInfo.targetLanguage }}"
    - step: REPLACE_XML
      at: CURSOR_PARAGRAPH
    - step: SHOW_NOTIFICATION
      message: "Content translated from {{ flowContext._dropInfo.sourceLanguage }} to {{ flowContext._dropInfo.targetLanguage }}"
```

#### Example: Process Dropped Images

Process images when dropped from external sources:

```yaml
- title: "Optimize Dropped Image"
  hidden: true
  swing:
    onBlockDrop:
      enable: true
      runWhen:
        blockType: "image"
        isCrossStory: true
  flow:
    - step: IF
      test: "{{ flowContext._dropInfo.referenceObject.path }}"
      then:
        - step: SHOW_NOTIFICATION
          message: "Processing image: {{ flowContext._dropInfo.referenceObject.path }}"
```

#### Example: Complex Conditions

Use the `when` expression for complex logic:

```yaml
- title: "Process Long Text Drops"
  hidden: true
  swing:
    onBlockDrop:
      enable: true
      runWhen:
        fromOldVersion: false
      when: |
        options.textContent && 
        options.textContent.length > 500 && 
        !options.isSameContainer
  flow:
    - step: HUB_COMPLETION
      instruction: "Summarize: {{ flowContext._dropInfo.textContent }}"
    - step: SHOW_RESPONSE
      title: "Summary"
```

#### Notes

- Block drop actions are typically configured with `hidden: true` since they are triggered automatically, not from the command palette
- Consider using `monitor.enabled: false` for quick actions to avoid UI interruption
- Multiple actions can register for block drop events - all matching actions will execute
- If conditions don't match, the action silently skips (no error shown)

### Combining Multiple Integrations

You can enable multiple integration types for the same action to make it accessible from different locations:

```yaml
- title: "AI Content Processor"
  description: "Process content with AI"
  swing:
    # Inline toolbar (for text selection in editor)
    inlineMenu:
      enable: true
      label: "Process"
      icon: "fa fa-cog"

    # Object action (for object three-dots menu)
    objectAction:
      enable: true

    # Context menu (for right-click)
    contextMenu:
      enable: true

  flow:
    - step: HUB_COMPLETION
      instruction: "Process and improve: {textContent}"

    # ...
```

This makes the action available in:
- Inline toolbar when text is selected
- Object three-dots menu in search/explorer
- Right-click context menu

### Migration from TEXT_MENU

The `TEXT_MENU` component is deprecated in v1.1.0. Migrate to `inlineMenu` for better integration with Swing.

**Old Approach (TEXT_MENU):**

```yaml
AI_KIT:
  TEXT_MENU:
    story:
      items:
        - type: 'button'
          icon: 'fa fa-compress'
          title: 'Shorten'
          flowRef: 'shortenText'

  SECTIONS:
    - section: Text Actions
      actions:
        - title: "shortenText"
          hidden: true
          flow:
            - step: HUB_COMPLETION
              instruction: "Shorten: {selectedText}"
            - step: REPLACE_TEXT
              at: CURSOR
```

**New Approach (inlineMenu):**

```yaml
AI_KIT:
  SECTIONS:
    - section: Text Actions
      actions:
        - title: "Shorten Text"
          swing:
            inlineMenu:
              enable: true
              icon: "fa fa-compress"
              label: "Shorten"
          flow:
            - step: HUB_COMPLETION
              instruction: "Shorten: {selectedText}"
            - step: REPLACE_TEXT
              at: CURSOR
```

**Benefits of the new approach:**
- No need for separate hidden actions
- Direct configuration in action definition
- Conditional enablement support
- Consistent with other Swing integrations
- Action remains accessible via command palette (Ctrl+K)

### Best Practices

#### 1. Use Descriptive Labels

Choose clear, action-oriented labels for inline toolbar buttons:

```yaml
# Good
swing:
  inlineMenu:
    label: "Improve Text"

# Avoid
swing:
  inlineMenu:
    label: "AI"
```

#### 2. Add Conditional Enablement

Prevent errors by enabling buttons only in valid contexts:

```yaml
swing:
  inlineMenu:
    enable: true
    label: "Format Paragraph"
    # Only enable for paragraphs with selected text
    isEnabled: "{{ ctx.activeDocument.getSelection().getNode().elementName == 'p' }}"
```

#### 3. Choose Appropriate Icons

Use FontAwesome icons that clearly represent the action:

```yaml
# Text improvement
icon: "fa fa-magic"

# Compression/shortening
icon: "fa fa-compress"

# Translation
icon: "fa fa-language"

# Generation/creation
icon: "fa fa-lightbulb"
```

#### 4. Combine Integrations for Consistency

Make frequently-used actions available in multiple places:

```yaml
- title: "Generate Headlines"
  swing:
    inlineMenu:
      enable: true
      icon: "fa fa-heading"
    objectAction:
      enable: true
    contextMenu:
      enable: true
```

#### 5. Handle Errors Gracefully

Use TRY/CATCH in flows to handle failures:

```yaml
flow:
  - step: TRY
    try:
      - step: HUB_COMPLETION
        instruction: "Process: {selectedText}"
      - step: REPLACE_TEXT
        at: CURSOR
    catch:
      - step: SHOW_NOTIFICATION
        message: "Processing failed. Please try again."
        type: "error"
```

#### 6. Test in All Contexts

Test your integrations in:
- Report editor
- Story editor
- DWP editor
- Search results
- Explorer view

#### 7. Respect Read-Only Mode

Set `allowReadOnly: false` for actions that modify content:

```yaml
swing:
  inlineMenu:
    enable: true
    allowReadOnly: false  # Disable in read-only mode
```

## UI Integration in Panels and Toolbars (Custom JavaScript)

ProActions can be triggered from various UI locations in Swing using the `SwingProActions.executeAction()` method.

### Executing Actions

The primary method for triggering ProActions from custom UI:

```javascript
SwingProActions.executeAction(ctx, actionId, options);
```

**Parameters:**

- `ctx` - The Swing context object (can be `null` or `undefined`, will auto-detect if not provided)
- `actionId` - The ID or title of the action to execute (from your configuration)
- `options` (optional) - Configuration object with the following properties:
  - `onChangeViewSize?: (width: number, height: number) => void` - Callback invoked when the view size changes. Primarily used by the `CHANGE_VIEW_SIZE` step to resize the command palette in Méthode Prime.
  - `onCompletion?: () => void` - Callback invoked when the action execution completes successfully
  - `onError?: (err: Error) => void` - Callback invoked when an error occurs during action execution
  - `onCancel?: (flowContext: ActionContextData) => void` - Callback invoked when the user cancels the flow (e.g., clicks cancel in FORM, PROMPT, FILE_UPLOAD, USER_SELECT, or IMAGE_PICKER steps)
  - `flowContext?: ActionContextData` - Prefilled context object that allows you to pass external data into the ProActions flow

#### Basic Usage

```javascript
// Simple execution
SwingProActions.executeAction(ctx, "my_action_id");
```

#### With Callbacks

```javascript
// With completion, error, and cancellation handling
SwingProActions.executeAction(ctx, "my_action_id", {
  onCompletion: () => {
    console.log("Action completed successfully");
    // Update UI or show success message
  },
  onError: (err) => {
    console.error("Action failed:", err);
    // Show error notification to user
  },
  onCancel: (flowContext) => {
    console.log("User cancelled the action");
    console.log("Flow state at cancellation:", flowContext);
    // Handle cancellation - no error notification needed
    // The flowContext contains the state of the workflow at the time of cancellation
  }
});
```

#### With Prefilled Context

Use `flowContext` to pass external data into your ProActions workflow:

```javascript
// Pass custom data to the flow
SwingProActions.executeAction(ctx, "process_article", {
  flowContext: {
    text: "Custom article text from external source",
    object: {
      category: "sports",
      priority: "high"
    },
    list: ["tag1", "tag2", "tag3"]
  },
  onCompletion: () => {
    console.log("Processing completed");
  }
});
```

This is useful when:
- Loading data from external APIs
- Pre-populating workflows with form data
- Chaining multiple actions with shared data
- Integrating with third-party systems

#### Understanding Cancellation vs Errors

ProActions distinguishes between **user cancellations** and **errors**:

- **Cancellation**: User explicitly cancels by clicking the cancel/close button in UI steps (FORM, PROMPT, FILE_UPLOAD, USER_SELECT, IMAGE_PICKER)
  - Triggers `onCancel` callback
  - Dispatches `aiKitFlowCancelled` custom event
  - Does NOT trigger `onError` callback
  - Not considered an error condition

- **Error**: Something goes wrong during flow execution (network error, invalid data, etc.)
  - Triggers `onError` callback
  - Dispatches `aiKitFlowError` custom event
  - Should display error notification to user

**Example: Handling both scenarios correctly**

```javascript
SwingProActions.executeAction(ctx, "process_article", {
  flowContext: {
    articleId: "123",
    mode: "summary"
  },

  onCompletion: () => {
    // Flow completed successfully
    showNotification("Article processed successfully", "success");
    refreshUI();
  },

  onCancel: (flowContext) => {
    // User cancelled - this is normal, not an error
    console.log("User cancelled processing");
    console.log("Cancelled at step:", flowContext);
    // Don't show error notification - cancellation is intentional
    // Optionally save draft state from flowContext
  },

  onError: (err) => {
    // Something went wrong - this is an actual error
    console.error("Processing failed:", err);
    showNotification("Failed to process article: " + err.message, "error");
  }
});
```

#### Listening to Custom Events

ProActions dispatches custom DOM events that you can listen to for global handling:

```javascript
// Listen for flow cancellations
document.addEventListener('aiKitFlowCancelled', (event) => {
  console.log('Flow cancelled:', {
    flowContext: event.detail.flowContext,
    cancelledAt: event.detail.cancelledAt,  // Step name where cancelled
    reason: event.detail.reason              // Always 'user_cancellation'
  });

  // Global cancellation handling
  // e.g., log analytics, save draft state, etc.
});

// Listen for flow completions
document.addEventListener('aiKitFlowCompleted', (event) => {
  console.log('Flow completed successfully');
  // Global completion handling
});

// Listen for flow errors
document.addEventListener('aiKitFlowError', (event) => {
  console.error('Flow error:', event.detail.error);
  // Global error handling
});
```

These events are useful for:
- Centralized logging and analytics
- Tracking user behavior patterns
- Implementing global error handling
- Saving draft states on cancellation
- Updating global UI indicators

### Toolbar Integration

Add ProActions buttons to the editor toolbar.

#### Step 1: Create a Custom Tab

First, create a custom tab to organize your ProActions buttons:

```javascript
eidosmedia.webclient.extensions.editor.tabs.add({
  name: "em-tab-proactions",
  label: "ProActions",
});
```

#### Step 2: Register Commands

Register commands that execute ProActions:

```javascript
eidosmedia.webclient.commands.add({
  name: "custom.proactions.robot.rewrite.pressrelease",
  isActive: function (ctx) {
    if (ctx.component.getType() === "toolbar") {
      return true;
    }
    return false;
  },
  isEnabled: function (ctx) {
    return true;
  },
  action: function (ctx, params, callbacks) {
    SwingProActions.executeAction(ctx, "robot.rewrite.pressrelease");
  },
});
```

#### Step 3: Add Toolbar Buttons

Add buttons to the toolbar that trigger your commands:

```javascript
eidosmedia.webclient.actions.editor.toolbar.addButton({
  action: "custom.proactions.robot.rewrite.pressrelease",
  label: "Rewrite (Press Release)",
  icon: "fas fa-robot",
  tabId: "em-tab-proactions",
});
```

#### Complete Example: Text-to-Speech Button

```javascript
// Create tab
eidosmedia.webclient.extensions.editor.tabs.add({
  name: "em-tab-proactions",
  label: "ProActions",
});

// Register command
eidosmedia.webclient.commands.add({
  name: "custom.proactions.tts",
  isActive: function (ctx) {
    if (ctx.component.getType() === "toolbar") {
      return true;
    }
    return false;
  },
  isEnabled: function (ctx) {
    return true;
  },
  action: function (ctx, params, callbacks) {
    SwingProActions.executeAction(ctx, "proactions.tts");
  },
});

// Add button
eidosmedia.webclient.actions.editor.toolbar.addButton({
  action: "custom.proactions.tts",
  label: "TTS",
  icon: "fas fa-podcast",
  tabId: "em-tab-proactions",
});
```

### Image Toolbar Integration

Add ProActions to the image component toolbar for context-aware image processing.

#### Register Image-Specific Commands

```javascript
eidosmedia.webclient.commands.add({
  name: "custom.proactions.vision.generatecaption",
  isActive: function (ctx) {
    if (ctx.area?.getName() === "story" &&
        ctx.component.getType() === ctx.activeDocument?.CONSTANTS?.BLOCK_IMAGE) {
      return true;
    }
    return false;
  },
  isEnabled: function (ctx) {
    if (ctx.component.getType() === ctx.activeDocument?.CONSTANTS?.BLOCK_IMAGE) {
      return true;
    }
    return false;
  },
  action: function (ctx, params, callbacks) {
    SwingProActions.executeAction(ctx, "vision_create_caption");
  },
});
```

#### Add Buttons to Image Toolbar

```javascript
eidosmedia.webclient.actions.editor.components.addButton({
  action: "custom.proactions.vision.generatecaption",
  label: "Generate Caption",
  icon: "fas fa-glasses",
  allowReadOnly: false,
});

eidosmedia.webclient.actions.editor.components.addButton({
  action: "custom.proactions.vision.describe.picture",
  label: "Describe Picture",
  icon: "fas fa-glasses",
  allowReadOnly: false,
});

eidosmedia.webclient.actions.editor.components.addButton({
  action: "custom.proactions.stability.outpaint",
  label: "Outpaint using AI",
  icon: "fas fa-glasses",
  allowReadOnly: false,
});
```

### Object Panel Integration

Integrate ProActions into custom object panels for document metadata processing.

#### Register Object Panel

```javascript
var types = [
  "EOM::CompoundStory",
  "EOM::Story",
  "WireStory",
  "EOM::CompoundMediaGallery",
  "EOM::MediaGallery",
  "EOM::Gallery",
  "EOM::LiveBlog",
  "Poll",
];

var options = {
  refreshOnShow: true,
  closeOnHide: true,

  init: function (ctx) {
    // Initialization logic
  },

  preFillForm: function (ctx) {
    console.log("ObjPanel: preFillForm");
  },

  postFillForm: function (ctx) {
    console.log("ObjPanel: postFillForm");

    // Add click handlers for ProActions buttons
    $("#getEntitiesOpenAI").click(function () {
      SwingProActions.executeAction(ctx, "text_extract_keywords");
    });

    $("#getSEOOpenAI").click(function () {
      SwingProActions.executeAction(ctx, "text_calc_seo_title");
      SwingProActions.executeAction(ctx, "text_calc_meta_descr");
    });
  },
};

eidosmedia.webclient.extensions.objectpanel.register(
  types,
  "story-objectpanel.html",
  options
);
```

#### Object Panel HTML (story-objectpanel.html)

Your HTML file would include buttons that trigger the ProActions:

```html
<div class="panel-section">
  <h3>AI Tools</h3>
  <button id="getEntitiesOpenAI" class="btn btn-primary">
    Extract Keywords
  </button>
  <button id="getSEOOpenAI" class="btn btn-primary">
    Generate SEO
  </button>
</div>
```

## API Integration

The ProActions API provides programmatic access to AI services and action execution.

### Getting the API Instance

```javascript
var aikit = SwingProActions.getAiKitApi();
```

### Available API Methods

#### 1. List Services

Get a list of all configured services:

```javascript
// List all services
const services = aikit.listServices();
console.log(services); // ["HUB", "OPENAI_COMPLETION", "DEEPL", ...]

// List services by type
const completionServices = aikit.listServices("completion");
console.log(completionServices); // ["HUB", "OPENAI_COMPLETION", "HUB_AZURE", ...]
```

**Parameters:**

- `type` (optional): Filter by service type (`"completion"`, `"speech"`, `"transcription"`, etc.)

**Returns:** Array of service names

#### 2. Get Configuration

Access the current ProActions configuration:

```javascript
const config = aikit.getConfig();
console.log(config.AI_KIT.SECTIONS);
console.log(config.AI_KIT.SERVICES);
```

**Returns:** The complete plugin configuration object

#### 3. Find Actions Using a Prompt

Find which actions use a specific prompt ID:

```javascript
const promptId = "c81b1538-d3da-47f1-836b-1158578a35d3";
const actionsUsingPrompt = aikit.getPromptUsingActions(promptId);
console.log(actionsUsingPrompt); // ["Replace Headline", "Suggest Headlines", ...]
```

**Parameters:**

- `promptId`: The UUID of the prompt to search for

**Returns:** Array of action titles or IDs that use the specified prompt

#### 4. Execute Chat Completion

Execute LLM chat completions programmatically:

##### Basic Usage

```javascript
const response = await aikit.executeChatCompletion("Hello, how are you?");
console.log(response.text); // The AI's response
```

##### With Options

```javascript
const response = await aikit.executeChatCompletion(
  "Summarize this article in 3 bullet points",
  {
    systemPrompt: "You are a helpful assistant for authors",
    serviceName: "HUB",
    completionType: "HUB_COMPLETION",
    includeFullResponse: true,
    model: "gpt-4o",
    ctx: currentSwingContext,
  }
);

console.log(response.text);        // The summary text
console.log(response.response);    // Full API response (if includeFullResponse: true)
```

**Parameters:**

- `userPrompt` (required): The user prompt string
- `options` (optional): Configuration object with the following properties:
  - `ctx`: Swing context object (auto-detected if not provided)
  - `systemPrompt`: System prompt to guide the AI's behavior
  - `serviceName`: Which service to use (default: `"HUB"`)
  - `completionType`: Which completion step to use (default: `"HUB_COMPLETION"`)
  - `includeFullResponse`: Include full API response (default: `false`)
  - `model`: Override the model (e.g., `"gpt-4o"`, `"gpt-4-turbo"`)
  - `flowContext`: Existing flow context to populate
  - `cfg`: Additional step configuration options

**Returns:** `ActionContextData` object containing:

- `text`: The main text response
- `object`: Parsed JSON object (if using structured outputs)
- `list`: Array of strings (if using list format)
- `response`: Full API response (if `includeFullResponse: true`)
- Other outputs depending on the completion configuration

##### Advanced Usage with Custom Configuration

```javascript
const response = await aikit.executeChatCompletion(
  "Extract key information from the article",
  {
    serviceName: "HUB",
    model: "gpt-4o",
    cfg: {
      response_format: "json_object",
      options: {
        temperature: 0.2,
        max_tokens: 500,
      },
    },
  }
);

const extractedData = response.object;
console.log(extractedData);
```

#### 5. Execute Action from YAML

Execute a complete ProActions workflow defined in YAML:

```javascript
const yamlDefinition = `
flow:
  - step: HUB_COMPLETION
    instruction: "Generate a headline for this article"
    behavior: "You are a helpful assistant"
  - step: REPLACE_TEXT
    at: XPATH
    xpath: "/doc/story/grouphead/headline/p"
`;

const result = await aikit.executeActionFromText(yamlDefinition);
console.log(result);
```

**Parameters:**

- `yamlStr`: YAML string defining the workflow

**Returns:** `ActionContextData` object with workflow results

### Practical Examples

#### Generate Keywords Button

```javascript
$("#generateKeywords").click(async function () {
  var aikit = SwingProActions.getAiKitApi();

  const result = await aikit.executeChatCompletion(
    "Extract 5-10 relevant keywords from this article content",
    {
      systemPrompt: "You are an SEO expert. Return only comma-separated keywords.",
      model: "gpt-4o-mini",
      ctx: ctx, // Swing Context
    }
  );

  // Update the keywords field
  $("#keywords-field").val(result.text);
});
```

## Best Practices

### 1. Error Handling

Always wrap API calls in try-catch blocks:

```javascript
$("#process-button").click(async function () {
  try {
    var aikit = SwingProActions.getAiKitApi();

    if (!aikit) {
      console.error("ProActions API not available");
      return;
    }

    const result = await aikit.executeChatCompletion("Process this content", {
      ctx: ctx,
    });

    // Handle success
    updateUI(result);
  } catch (error) {
    console.error("ProActions execution failed:", error);
    // Show user-friendly error message
    showErrorNotification("AI processing failed. Please try again.");
  }
});
```

### 2. Context Management

Ensure the Swing context is properly passed:

```javascript
// In object panel
postFillForm: function (ctx) {
  $("#aiButton").click(function () {
    // Pass the ctx from the closure
    SwingProActions.executeAction(ctx, "my_action");
  });
}
```

### 3. Action ID Naming

Use consistent, descriptive action IDs:

```yaml
# Good
- title: "Generate SEO Title"
  id: "text_calc_seo_title"

# Avoid
- title: "SEO"
  id: "action1"
```

### 4. Service Configuration

Check service availability before using specific services:

```javascript
const services = aikit.listServices("completion");
const serviceName = services.includes("OPENAI_COMPLETION")
  ? "OPENAI_COMPLETION"
  : "HUB";

const result = await aikit.executeChatCompletion(prompt, {
  serviceName: serviceName,
});
```

### 5. Loading States

Show loading indicators during API calls:

```javascript
$("#process-button").click(async function () {
  const $button = $(this);
  $button.prop("disabled", true).text("Processing...");

  try {
    const result = await aikit.executeChatCompletion("...", { ctx: ctx });
    updateUI(result);
  } catch (error) {
    handleError(error);
  } finally {
    $button.prop("disabled", false).text("Process");
  }
});
```

## Troubleshooting

### Common Issues

**Issue**: `SwingProActions is not defined`

- Ensure ProActions extension is loaded before your custom scripts
- Check browser console for loading errors

**Issue**: Action doesn't execute

- Verify the action ID matches exactly (case-sensitive)
- Check that the action is defined in `pro-actions.ai-kit.yaml`
- Ensure the context is valid

**Issue**: API returns undefined

- Check that the extension is loaded: `SwingProActions.extensions.get("pro-actions-ai-kit")`
- Verify the service is configured in the services section

**Issue**: Toolbar buttons don't appear

- Verify tab creation code runs before button registration
- Check command names match between command registration and button configuration
- Clear browser cache

### Debug Tips

Enable verbose logging:

```javascript
// Check if ProActions is loaded
console.log("ProActions available:", typeof SwingProActions !== "undefined");

// Check if API is accessible
var aikit = SwingProActions.getAiKitApi();
console.log("API available:", !!aikit);

// List available services
if (aikit) {
  console.log("Available services:", aikit.listServices());
}
```

## Security Considerations

### 1. Input Validation

Always validate user input before passing to AI services:

```javascript
const userInput = $("#user-field").val().trim();
if (!userInput || userInput.length > 10000) {
  showError("Invalid input");
  return;
}

const result = await aikit.executeChatCompletion(userInput, {
  ctx: ctx,
});
```

### 2. Rate Limiting

Implement client-side rate limiting for repeated calls:

```javascript
let lastCallTime = 0;
const MIN_INTERVAL = 2000; // 2 seconds

$("#ai-button").click(async function () {
  const now = Date.now();
  if (now - lastCallTime < MIN_INTERVAL) {
    showError("Please wait before making another request");
    return;
  }
  lastCallTime = now;

  // Proceed with API call
});
```

### 3. Error Messages

Don't expose internal errors to users:

```javascript
try {
  const result = await aikit.executeChatCompletion(prompt, { ctx: ctx });
} catch (error) {
  console.error("Internal error:", error); // Log for debugging
  showError("Processing failed. Please try again."); // User-friendly message
}
```

## Further Reading

- **[Configuration Basics](./basics.md)** - ProActions configuration fundamentals
- **[Service Configuration](./service-configuration.md)** - Configure AI services
- **[AI Integration Guide](./ai-integration.md)** - Detailed LLM integration
- **[Workflow Design](./workflow-design.md)** - Best practices for workflows
- **[Step Library](../../step-library/overview.md)** - Available workflow steps
