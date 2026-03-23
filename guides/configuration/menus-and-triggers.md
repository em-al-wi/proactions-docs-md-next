# Menus and Triggers

ProActions provides multiple ways to trigger workflows beyond the command palette (Ctrl+K). This guide covers text selection menus, one-click menus, instant actions, and programmatic execution.

## Overview of Trigger Methods

ProActions workflows can be triggered through:

1. **Native Swing Integrations** (v1.1.0+) - Inline toolbar, object actions, context menus
2. **Command Palette** (Ctrl+K) - Search and execute actions
3. **Slash Command Menu** - Notion-style slash commands in the editor
4. **One-Click Menu** - Floating menu for quick actions
5. **Text Selection Menu** - Context menu when selecting text (deprecated from v1.1.0)
6. **Instant Actions** - Execute based on cursor position
7. **Programmatic Execution** - Called from custom code
8. **Event-Driven Triggers** - Execute automatically when subscribed events are emitted

## Native Swing Integrations (v1.1.0+)

Starting with v1.1.0, ProActions supports declarative integration with Swing's native UI components. For comprehensive documentation on these features, see:

**[Swing Integration Guide](./swing-integration.md#native-swing-integrations-v110)** - Complete reference for:

- **inlineMenu** - Add buttons to the inline text editor toolbar
- **objectAction** - Add actions to object three-dots menu
- **contextMenu** - Add items to right-click context menus

These integrations are configured directly in your action definitions using the `swing:` property and offer:
- Declarative configuration (no custom JavaScript required)
- Conditional visibility and enablement via expressions
- Consistent integration with Swing's native UI
- Support for FontAwesome icons and custom labels

**Quick Example:**

```yaml
- title: "Improve Selection"
  swing:
    inlineMenu:
      enable: true
      label: "Improve"
      icon: "fa fa-magic"
      isEnabled: "{{ client.getObjectType() == 'EOM::CompoundStory' }}"
  flow:
    - step: HUB_COMPLETION
      instruction: "Improve: {selectedText}"
    - step: REPLACE_TEXT
      at: CURSOR
```

For complete documentation with all configuration options, examples, best practices, and troubleshooting, see the [Swing Integration Guide](./swing-integration.md#native-swing-integrations-v110).

## Event-Driven Triggers

Use action-level `events` subscriptions to trigger actions automatically from lifecycle or custom events.

For a compact lifecycle reference (`aiKitReady`, `aiKitFlowCompleted`, and usage patterns), see the
[AI-Kit Lifecycle Event Reference](./swing-integration.md#ai-kit-lifecycle-event-reference).

```yaml
- id: auto_follow_up
  title: "Auto Follow-Up"
  hidden: true
  events:
    - source: "aikit"
      name: "aiKitFlowCompleted"
      condition: "{{ flowContext._eventPayload.actionId == 'source-action' }}"
      ifRunning: "queue"

  flow:
    - step: SHOW_NOTIFICATION
      message: "Follow-up action executed"
```

### Trigger Policies (`ifRunning`)

- `skip` - Ignore new trigger while action is running
- `queue` - Queue trigger and execute after the current run
- `parallel` - Execute immediately in parallel

### Event Data In Triggered Flows

When an action is started by an event subscription, AI-Kit injects event data into `flowContext`:

- `flowContext._event` - Full normalized event object
- `flowContext._eventName` - Event name
- `flowContext._eventPayload` - Event payload (if provided)

Example:

```yaml
- id: auto_follow_up
  title: "Auto Follow-Up"
  hidden: true
  events:
    - source: "aikit"
      name: "aiKitFlowCompleted"
      condition: "{{ flowContext._eventPayload.actionId == 'source-action' }}"

  flow:
    - step: IF
      condition: "{{ flowContext._eventPayload.actionId == 'source-action' }}"
      then:
        - step: SHOW_NOTIFICATION
          message: "Follow-up for {{ flowContext._eventPayload.actionId }}"
```

`events.condition` uses the standard resolver boolean semantics (same as other YAML conditions). Reference event data through `flowContext._event`, `flowContext._eventName`, and `flowContext._eventPayload`.

### Emitting Custom Events from Flows

Use `EMIT_EVENT` (alias `DISPATCH_EVENT`) to publish custom events:

```yaml
- step: EMIT_EVENT
  name: "custom.contentProcessed"
  payload:
    summary: "{{ flowContext.text }}"
```

Other actions can subscribe with `events.source: custom` and matching `events.name`.

## Slash Command Menu

The slash command menu provides a Notion-style command interface that appears when you type `/` in the editor. It offers context-aware actions with real-time filtering as you type.

### Configuration Structure

**File:** `/SysConfig/ProActions/slash.ai-kit.menu.yaml`

```yaml
story:
  items:
    - type: 'button'
      icon: 'fa fa-icon'
      title: 'Command Name'
      flowRef: 'action_id'
      forElement:
        - elementType
      forContentItem:
        - contentItemType
```

### Basic Slash Menu

```yaml
story:
  items:
    - type: 'button'
      icon: 'fa fa-heading'
      title: 'H1 Heading'
      flowRef: 'text.format.h1'
      forElement:
        - p

    - type: 'button'
      icon: 'fa fa-image'
      title: 'Insert Picture'
      flowRef: 'text.insert.picture'
      forElement:
        - p

    - type: 'separator'

    - type: 'button'
      icon: 'fa fa-magic'
      title: 'Generate Subheadline'
      flowRef: 'text.add.subheadline'
      forElement:
        - p
      forContentItem:
        - text
```

### Menu Item Properties

| Property         | Description            | Required | Example             |
| ---------------- | ---------------------- | -------- | ------------------- |
| `type`           | Item type              | Yes      | `'button'`          |
| `icon`           | FontAwesome icon       | No       | `'fa fa-heading'`   |
| `title`          | Command label          | Yes      | `'H1 Heading'`      |
| `flowRef`        | Action ID              | Yes      | `'text.format.h1'`  |
| `forElement`     | Element types          | No       | `['p', 'h1']`       |
| `forContentItem` | Content item types     | No       | `['text', 'body']`  |

### Using Separators

```yaml
story:
  items:
    - type: 'button'
      title: 'H1 Heading'
      flowRef: 'text.format.h1'

    - type: 'button'
      title: 'H2 Heading'
      flowRef: 'text.format.h2'

    - type: 'separator'  # Visual separator

    - type: 'button'
      title: 'Insert Picture'
      flowRef: 'text.insert.picture'
```

### Including Slash Menu in Main Configuration

```yaml
# /SysConfig/pro-actions.ai-kit.yaml
AI_KIT:
  SERVICES:
    !include /SysConfig/ProActions/services.ai-kit.services.yaml

  SLASH_MENU:
    !include /SysConfig/ProActions/slash.ai-kit.menu.yaml

  SECTIONS:
    - !include /SysConfig/ProActions/sections/slash-actions.yaml
```

### How It Works

1. **Trigger**: Type `/` in an empty or existing paragraph
2. **Filter**: Continue typing to filter commands (e.g., `/head` shows heading options)
3. **Navigate**: Use arrow keys or Tab to navigate options
4. **Execute**: Press Enter to execute the selected command
5. **Cancel**: Press Escape or Space to close the menu

The slash command automatically removes the `/command` text when executed.

## One-Click Menu

The one-click menu is a floating menu that appears for specific content elements, providing quick access to contextual actions.

### Configuration Structure

**File:** `/SysConfig/ProActions/menu.oneclick.yaml`

```yaml
story:
  items:
    - type: 'button'
      icon: 'fa fa-icon'
      title: 'Button Label'
      flowRef: 'action_id'
      forContentItem:
        - elementName
      forElement:
        - elementType
```

### Basic One-Click Menu

```yaml
story:
  items:
    - type: 'button'
      icon: 'fa fa-list'
      title: 'Suggest headlines'
      flowRef: 'suggest_headlines'
      forContentItem:
        - grouphead
      forElement:
        - p
        - dummyText
```

### Menu Item Properties

| Property         | Description        | Required | Example                 |
| ---------------- | ------------------ | -------- | ----------------------- |
| `type`           | Item type          | Yes      | `'button'`              |
| `icon`           | FontAwesome icon   | No       | `'fa fa-magic'`         |
| `title`          | Button label       | Yes      | `'Generate'`            |
| `flowRef`        | Action ID          | Yes      | `'generateHeadline'`    |
| `forContentItem` | Content item types | No       | `['grouphead', 'body']` |
| `forElement`     | Element types      | No       | `['p', 'h1']`           |

### Element-Specific One-Click Menus

**For Headlines:**

```yaml
story:
  items:
    - type: 'button'
      icon: 'fa fa-lightbulb'
      title: 'Suggest headlines'
      flowRef: 'suggest_headlines'
      forContentItem:
        - grouphead
      forElement:
        - p
        - dummyText
```

**For Body Paragraphs:**

```yaml
story:
  items:
    - type: 'button'
      icon: 'fa fa-paragraph'
      title: 'Improve paragraph'
      flowRef: 'improve_paragraph'
      forContentItem:
        - body
      forElement:
        - p
```

### Including One-Click Menu in Main Configuration

```yaml
# /SysConfig/pro-actions.ai-kit.yaml
AI_KIT:
  SERVICES:
    !include /SysConfig/ProActions/services.ai-kit.services.yaml

  ONECLICK_MENU:
    !include /SysConfig/ProActions/menu.oneclick.yaml

  SECTIONS:
    - !include /SysConfig/ProActions/sections/headlines.yaml
```

## Text Selection Menu

:warning: The text selection menu is deprecated from v1.1.0.

:::danger Deprecated
The TEXT_MENU is deprecated from v1.1.0 and will be removed in v2.0.0. 
Migrate to `swing.inlineMenu` (see [Native Swing Integrations](#native-swing-integrations-v110)).
:::

### Migration Guide: TEXT_MENU to inlineMenu

**Before (TEXT_MENU - Deprecated):**

```yaml
# text.ai-kit.menu.yaml
story:
  items:
  - type: 'button'
    icon: 'fa fa-magic'
    title: 'Improve'
    flowRef: 'improveSelection'

# Main configuration
AI_KIT:
  TEXT_MENU:
    !include /SysConfig/ProActions/text.ai-kit.menu.yaml

SECTIONS:
- section: Text Actions
  actions:
  - title: 'improveSelection'
    hidden: true  # Hidden from Ctrl+K
    flow:
    - step: HUB_COMPLETION
      instruction: "Improve: {selectedText}"
    - step: REPLACE_TEXT
      at: CURSOR
```

**After (inlineMenu - Recommended):**

```yaml
# No separate menu file needed!

# Main configuration
AI_KIT:
  SECTIONS:
  - section: Text Actions
    actions:
    - title: "Improve Selection"
      swing:
        inlineMenu:
          enable: true
          label: "Improve"
          icon: "fa fa-magic"
          isEnabled: "{{ ctx.activeDocument.getSelection().getNode().elementName == 'p' }}"
      flow:
      - step: HUB_COMPLETION
        instruction: "Improve: {selectedText}"
      - step: REPLACE_TEXT
        at: CURSOR
```

**Benefits of migration:**
- ✅ No separate menu configuration file needed
- ✅ Direct action configuration in one place
- ✅ Conditional visibility with `isEnabled`
- ✅ Consistent with other Swing integrations
- ✅ Action remains accessible via Ctrl+K (no need for `hidden: true`)

**Migration steps:**
1. Remove the `TEXT_MENU` include from your main configuration
2. For each menu item, add `swing.inlineMenu` configuration to the action
3. Remove `hidden: true` if you want the action in Ctrl+K
4. Add conditional `isEnabled` expressions for proper context
5. Test in Swing to verify buttons appear correctly
6. Delete the old `text.ai-kit.menu.yaml` file

### Configuration Structure

**File:** `/SysConfig/ProActions/text.ai-kit.menu.yaml`

```yaml
story:
  items:
  - type: 'button'
    icon: 'fa fa-icon'
    title: 'Button Label'
    flowRef: 'action_id'
```

### Basic Text Menu

```yaml
story:
  items:
    - type: 'button'
      icon: 'fa fa-redo'
      title: 'Rewrite'
      flowRef: 'rephraseSelection'

    - type: 'button'
      icon: 'fa fa-compress'
      title: 'Shorten'
      flowRef: 'shortenSelection'

    - type: 'button'
      icon: 'fa fa-language'
      title: 'Translate'
      flowRef: 'translateSelection'
```

### Menu Item Properties

| Property  | Description             | Required | Example          |
| --------- | ----------------------- | -------- | ---------------- |
| `type`    | Item type               | Yes      | `'button'`       |
| `icon`    | FontAwesome icon class  | No       | `'fa fa-edit'`   |
| `title`   | Button label            | Yes      | `'Improve Text'` |
| `flowRef` | ID of action to execute | Yes      | `'improveText'`  |

### Creating Hidden Actions for Menus

Actions referenced by menus can be hidden from the command palette:

```yaml
section: Text Menu Actions
actions:
  - title: 'rephraseSelection'
    hidden: true  # Don't show in Ctrl+K menu
    flow:
      - step: HUB_COMPLETION
        behavior: |
          You are an AI assistant helping authors.
          Rephrase the given text while maintaining meaning.
        instruction: 'Rephrase: {selectedText}'

      - step: REPLACE_TEXT
        at: CURSOR

  - title: 'shortenSelection'
    hidden: true
    flow:
      - step: HUB_COMPLETION
        instruction: 'Shorten this text: {selectedText}'

      - step: REPLACE_TEXT
        at: CURSOR
```

### Including Text Menu in Main Configuration

```yaml
# /SysConfig/pro-actions.ai-kit.yaml
AI_KIT:
  SERVICES:
    !include /SysConfig/ProActions/services.ai-kit.services.yaml

  TEXT_MENU:
    !include /SysConfig/ProActions/text.ai-kit.menu.yaml

  SECTIONS:
    - !include /SysConfig/ProActions/sections/text-menu-actions.yaml
```

### Complete Text Menu Example

**Menu Configuration:**

```yaml
# /SysConfig/ProActions/text.ai-kit.menu.yaml
story:
  items:
    - type: 'button'
      icon: 'fa fa-magic'
      title: 'Improve'
      flowRef: 'improveSelection'

    - type: 'button'
      icon: 'fa fa-redo'
      title: 'Rewrite'
      flowRef: 'rewriteSelection'

    - type: 'button'
      icon: 'fa fa-compress'
      title: 'Shorten'
      flowRef: 'shortenSelection'

    - type: 'button'
      icon: 'fa fa-expand'
      title: 'Expand'
      flowRef: 'expandSelection'
```

**Actions Configuration:**

```yaml
# /SysConfig/ProActions/sections/text-menu-actions.yaml
section: Text Menu Actions
actions:
  - title: 'improveSelection'
    hidden: true
    flow:
      - step: HUB_COMPLETION
        instruction: 'Improve: {selectedText}'
      - step: REPLACE_TEXT
        at: CURSOR

  - title: 'rewriteSelection'
    hidden: true
    flow:
      - step: HUB_COMPLETION
        instruction: 'Rewrite: {selectedText}'
      - step: REPLACE_TEXT
        at: CURSOR

  - title: 'shortenSelection'
    hidden: true
    flow:
      - step: HUB_COMPLETION
        instruction: 'Shorten: {selectedText}'
      - step: REPLACE_TEXT
        at: CURSOR

  - title: 'expandSelection'
    hidden: true
    flow:
      - step: HUB_COMPLETION
        instruction: 'Expand on: {selectedText}'
      - step: REPLACE_TEXT
        at: CURSOR
```

## Instant Actions

Instant actions execute automatically based on cursor position when the instant action keyboard shortcut is pressed (Ctrl+Shift+K).

### Configuring Instant Actions

```yaml
- title: 'Quick Improve'
  instantActionContext: "/doc/story/body/p,/doc/story/headline/p"
  flow:
    - step: HUB_COMPLETION
      instruction: "Improve: {selectedText}"

    - step: REPLACE_TEXT
      at: CURSOR
```

### Instant Action Properties

| Property               | Description                | Example                                     |
| ---------------------- | -------------------------- | ------------------------------------------- |
| `instantActionContext` | Comma-separated XPath list | `"/doc/story/body/p,/doc/story/headline/p"` |

### XPath Context Examples

**Body paragraphs only:**

```yaml
instantActionContext: "/doc/story/body/p"
```

**Headlines and body:**

```yaml
instantActionContext: "/doc/story/headline/p,/doc/story/body/p"
```

**Multiple content areas:**

```yaml
instantActionContext: "/doc/story/body/p,/doc/story/standfirst/p,/doc/story/summary/p"
```

## Programmatic Execution

Execute ProActions workflows programmatically from custom code.

### Execute Action by ID

```javascript
// Execute a hidden action
SwingProActions.executeAction(ctx, "myActionId");
```

### Custom Button Integration

Create custom toolbar buttons that trigger ProActions:

```javascript
// Register custom command
eidosmedia.webclient.commands.add({
  name: "custom.myProAction",
  label: "My ProAction",
  isActive: function(ctx) { return true; },
  isEnabled: function(ctx) {
    // Only enable for specific elements
    if (ctx.component.getType() === ctx.activeDocument.CONSTANTS.BLOCK_IMAGE) {
      return true;
    }
    return false;
  },
  action: function(ctx, params, callbacks) {
    SwingProActions.executeAction(ctx, "processImage");
  }
});

// Add button to toolbar
eidosmedia.webclient.actions.editor.components.addButton({
  action: "custom.myProAction",
  label: "Process Image",
  icon: "fas fa-magic",
  allowReadOnly: false
});
```

## Complete Integration Example

### File Structure

```
/SysConfig/
├── pro-actions.ai-kit.yaml
└── ProActions/
    ├── services.ai-kit.services.yaml
    ├── slash.ai-kit.menu.yaml
    ├── text.ai-kit.menu.yaml
    ├── menu.oneclick.yaml
    └── sections/
        ├── content-tools.yaml
        └── menu-actions.yaml
```

### Main Configuration

```yaml
# /SysConfig/pro-actions.ai-kit.yaml
AI_KIT:
  SERVICES:
    !include /SysConfig/ProActions/services.ai-kit.services.yaml

  SLASH_MENU:
    !include /SysConfig/ProActions/slash.ai-kit.menu.yaml

  TEXT_MENU:
    !include /SysConfig/ProActions/text.ai-kit.menu.yaml

  ONECLICK_MENU:
    !include /SysConfig/ProActions/menu.oneclick.yaml

  SECTIONS:
    - !include /SysConfig/ProActions/sections/content-tools.yaml
    - !include /SysConfig/ProActions/sections/menu-actions.yaml
```

### Slash Menu Configuration

```yaml
# /SysConfig/ProActions/slash.ai-kit.menu.yaml
story:
  items:
    - type: 'button'
      icon: 'fa fa-heading'
      title: 'H1 Heading'
      flowRef: 'text.format.h1'
      forElement:
        - p

    - type: 'button'
      icon: 'fa fa-image'
      title: 'Insert Picture'
      flowRef: 'text.insert.picture'
      forElement:
        - p
```

### Text Menu Configuration

```yaml
# /SysConfig/ProActions/text.ai-kit.menu.yaml
story:
  items:
    - type: 'button'
      icon: 'fa fa-magic'
      title: 'Improve'
      flowRef: 'improveText'

    - type: 'button'
      icon: 'fa fa-language'
      title: 'Translate to French'
      flowRef: 'translateFR'
```

### One-Click Menu Configuration

```yaml
# /SysConfig/ProActions/menu.oneclick.yaml
story:
  items:
    - type: 'button'
      icon: 'fa fa-lightbulb'
      title: 'Suggest headlines'
      flowRef: 'suggest_headlines'
      forContentItem:
        - grouphead
      forElement:
        - p
        - dummyText
```

### Menu Actions

```yaml
# /SysConfig/ProActions/sections/menu-actions.yaml
section: Menu Actions
actions:
  - title: 'improveText'
    hidden: true
    flow:
      - step: HUB_COMPLETION
        instruction: "Improve: {selectedText}"
      - step: REPLACE_TEXT
        at: CURSOR

  - title: 'translateFR'
    hidden: true
    flow:
      - step: DEEPL_TRANSLATE
        instruction: "{selectedText}"
        target_lang: "FR"
      - step: REPLACE_TEXT
        at: CURSOR

  - title: 'suggest_headlines'
    hidden: true
    flow:
      - step: HUB_COMPLETION
        instruction: "Generate 5 headlines for: {textContent}"
        response_format: "list"
      - step: USER_SELECT
        promptText: "Choose a headline:"
      - step: REPLACE_TEXT
        at: XPATH
        xpath: "/doc/story/grouphead/headline/p"
```

## Best Practices

### 1. Hide Menu-Only Actions

```yaml
# Good - hidden from Ctrl+K
- title: 'improveSelection'
  hidden: true
  flow:
    # steps...

# Avoid - shows in both menu and Ctrl+K
- title: 'Improve Selection'
  flow:
    # steps...
```

### 2. Use Clear, Action-Oriented Labels

```yaml
# Good
- title: 'Translate to French'
  icon: 'fa fa-language'

# Avoid
- title: 'French'
  icon: 'fa fa-flag'
```

### 3. Group Related Menu Items

```yaml
story:
  items:
    # Text improvements
    - type: 'button'
      title: 'Improve'
      flowRef: 'improve'
    - type: 'button'
      title: 'Rewrite'
      flowRef: 'rewrite'

    - type: 'separator'  # Use separators in slash menu

    # Translations
    - type: 'button'
      title: 'To French'
      flowRef: 'translateFR'
    - type: 'button'
      title: 'To German'
      flowRef: 'translateDE'
```

### 4. Use Appropriate Icons

```yaml
# FontAwesome icons
- icon: 'fa fa-magic'       # AI/improvement
- icon: 'fa fa-language'    # Translation
- icon: 'fa fa-compress'    # Shorten
- icon: 'fa fa-expand'      # Expand
- icon: 'fa fa-redo'        # Rewrite
- icon: 'fa fa-lightbulb'   # Suggestions
```

### 5. Scope Menus Appropriately

```yaml
# Good - specific to headlines (one-click menu)
- title: 'Suggest headlines'
  flowRef: 'headlines'
  forContentItem:
    - grouphead
  forElement:
    - p

# Good - context-aware (slash menu)
- title: 'Insert Picture'
  flowRef: 'insert.picture'
  forElement:
    - p  # Only show in paragraphs

# Avoid - shows everywhere
- title: 'Suggest headlines'
  flowRef: 'headlines'
```

### 6. Use Descriptive Titles for Slash Commands

```yaml
# Good - clear and searchable
- title: 'H1 Heading'
- title: 'Insert Picture'
- title: 'Generate Subheadline'

# Avoid - too short or ambiguous
- title: 'H1'
- title: 'Pic'
- title: 'Gen'
```

## Troubleshooting

### Slash menu doesn't appear

**Check:**

- You're typing `/` in a paragraph element
- Configuration is included in main YAML file
- Cursor is in a supported element type

**Solution:**

- Verify inclusion: `!include /SysConfig/ProActions/slash.ai-kit.menu.yaml`
- Check `forElement` matches your cursor location
- Review browser console for errors

### Slash menu shows "No results"

**Check:**

- Command title matches your filter text
- Context criteria (`forElement`, `forContentItem`) match current location
- User permissions allow access to the command

**Solution:**

- Try typing just `/` to see all available commands
- Check `forElement` and `forContentItem` configuration
- Verify user is in correct group/team

### Menu doesn't appear

**Check:**

- Configuration is included in main YAML file
- Syntax is valid
- File permissions are correct

**Solution:**

- Verify inclusion: `!include /SysConfig/ProActions/oneclick.ai-kit.menu.yaml`
- Check browser console for errors
- Refresh configuration cache

### Action doesn't execute

**Check:**

- `flowRef` matches action title
- Action is defined in sections
- Action flow is valid

**Solution:**

- Verify action ID matches exactly
- Check action is not disabled
- Review browser console for errors

### Instant action not working

**Check:**

- XPath is correct for your document structure
- Cursor is in specified context
- Instant action shortcut is configured

**Solution:**

- Test XPath in browser console
- Verify keyboard shortcut in XsmileKbd.cfg
- Check `instantActionContext` syntax

## Next Steps

- **[Configuration Basics](./basics.md)** - Overall configuration concepts
- **[Keyboard Shortcuts](../../setup/keyboard-shortcuts.md)** - Configure shortcuts
