# Keyboard Shortcuts

ProActions provides keyboard shortcuts for quick access to workflows and automation features. This guide explains how to configure and use keyboard shortcuts in Swing and Prime.

## Default Shortcuts

ProActions comes with these default shortcuts:

| Shortcut                                               | Command         | Description                                             |
| ------------------------------------------------------ | --------------- | ------------------------------------------------------- |
| `Ctrl+K` (Windows/Linux)<br/>`Cmd+K` (Mac)             | Open ProActions | Opens the command palette to search and execute actions |
| `Ctrl+Shift+K` (Windows/Linux)<br/>`Cmd+Shift+K` (Mac) | Instant Action  | Executes context-specific instant action (optional)     |

## Editor Context Shortcuts (Recommended)

The recommended way to configure ProActions shortcuts is through Swing's keyboard configuration file. This provides the most reliable integration with the editor context.

### Configuration File

**File:** `/SysConfig/XsmileKbd.cfg`

### Basic Configuration

Add this configuration within the `<context type="Text">` section:

```xml
<context type="Text">
  <!-- ... other keyboard shortcuts ... -->

  <key description="K" charCode="75">
    <modifier type="C" shortcut="Ctrl+K">
      <function name="editorExtension" command="custom.openSwingProActions"/>
    </modifier>
  </key>

  <!-- ... other keyboard shortcuts ... -->
</context>
```

### With Instant Action

To add the instant action shortcut:

```xml
<context type="Text">
  <!-- ... other keyboard shortcuts ... -->

  <key description="K" charCode="75">
    <!-- Open ProActions command palette -->
    <modifier type="C" shortcut="Ctrl+K">
      <function name="editorExtension" command="custom.openSwingProActions"/>
    </modifier>

    <!-- Instant action (context-specific) -->
    <modifier type="CS" shortcut="Ctrl+Shift+K">
      <function name="editorExtension" command="custom.instantSwingProActions"/>
    </modifier>
  </key>

  <!-- ... other keyboard shortcuts ... -->
</context>
```

### Enhanced Configuration for Newer Swing Versions

In newer Swing versions (6.2025.06 and later), a new shortcut type `customSwingExtension` is available that enables ProActions to work in **all contexts**, including the dashboard:

```xml
<context type="all">
  <!-- ... other keyboard shortcuts ... -->

  <key description="K" charCode="75">
    <!-- Open ProActions in all contexts (dashboard, editor, etc.) -->
    <modifier type="C" shortcut="Ctrl+K">
      <function name="customSwingExtension" command="custom.openSwingProActions"/>
    </modifier>

    <!-- Instant action (editor context only) -->
    <modifier type="CS" shortcut="Ctrl+Shift+K">
      <function name="editorExtension" command="custom.instantSwingProActions"/>
    </modifier>
  </key>

  <!-- ... other keyboard shortcuts ... -->
</context>
```

**Key differences:**

- **Context**: `type="all"` instead of `type="Text"` - enables shortcuts everywhere
- **Function**: `customSwingExtension` instead of `editorExtension` - works in dashboard and other contexts
- **Instant actions**: Still use `editorExtension` as they only work in editor context

**Benefits of the new configuration:**

- ✅ ProActions accessible from dashboard, reports, and other Swing contexts
- ✅ Consistent keyboard shortcut across all areas of Swing
- ✅ Better user experience for dashboard-based workflows

**Note:** For older Swing versions, keep using the `editorExtension` configuration shown above and the fallback shortcuts for the dashboard shown below.

### Custom Key Combinations

You can change the key combination to suit your preferences:

```xml
<!-- Use Ctrl+Space instead of Ctrl+K -->
<key description="Space" charCode="32">
  <modifier type="C" shortcut="Ctrl+Space">
    <function name="editorExtension" command="custom.openSwingProActions"/>
  </modifier>
</key>
```

```xml
<!-- Use Ctrl+P for ProActions -->
<key description="P" charCode="80">
  <modifier type="C" shortcut="Ctrl+P">
    <function name="editorExtension" command="custom.openSwingProActions"/>
  </modifier>
</key>
```

### Common Character Codes

| Key   | Char Code | Description  |
| ----- | --------- | ------------ |
| K     | 75        | Letter K     |
| P     | 80        | Letter P     |
| Space | 32        | Spacebar     |
| Enter | 13        | Enter/Return |

## Fallback Shortcuts

For contexts where editor shortcuts are not available (like the Swing dashboard), ProActions provides a fallback shortcut mechanism.

### Configuration

**File:** `~/methode-servlets/conf/swing/com.eidosmedia.swing.web-app/app/plugins/pro-actions/pro-actions-config.js`

```javascript
(() => {
  window.SwingProActionsConfig = {
    // Fallback keyboard shortcut
    // Uses hotkeys-js syntax: https://github.com/jaywcjlove/hotkeys-js
    openHotkey: 'cmd+k,ctrl+k',

    // ... other configuration ...
  };
})();
```

The `openHotkey` property uses [hotkeys-js](https://github.com/jaywcjlove/hotkeys-js) syntax:

**Key combinations:**

```javascript
openHotkey: 'ctrl+k'      // Ctrl+K
openHotkey: 'cmd+k'       // Cmd+K (Mac)
openHotkey: 'alt+p'       // Alt+P
openHotkey: 'shift+space' // Shift+Space
```

**Supported modifier keys:**

- `ctrl` - Control key
- `cmd` / `command` - Command key (Mac)
- `shift` - Shift key
- `alt` / `option` - Alt/Option key
- `meta` - Meta key

**Supported special keys:**

- `space`, `enter`, `tab`, `backspace`
- `escape`, `delete`
- `up`, `down`, `left`, `right`
- `f1` through `f12`
- And many more...

## Instant Actions (Swing only!)

Instant actions allow you to execute specific actions based on the current cursor context without opening the command palette.

### Configuring Instant Actions

In your action configuration, use the `instantActionContext` property:

```yaml
- title: 'Improve Selected Text'
  instantActionContext: "/doc/story/body/p,/doc/story/headline/p"
  flow:
    - step: HUB_COMPLETION
      instruction: "Improve this text: {selectedText}"

    - step: REPLACE_TEXT
      at: CURSOR
```

### XPath Context Specification

The `instantActionContext` property accepts a comma-separated list of XPath expressions:

```yaml
# Single context
instantActionContext: "/doc/story/body/p"

# Multiple contexts
instantActionContext: "/doc/story/body/p,/doc/story/headline/p,/doc/story/standfirst/p"
```

When the user presses the instant action shortcut (`Ctrl+Shift+K`), ProActions:

1. Checks the current cursor position
2. Matches it against configured instant action contexts
3. Executes the matching action automatically

## Keyboard Shortcut Best Practices

### 1. Avoid Conflicts

Check for conflicts with existing Swing/Prime shortcuts:

```xml
<!-- Common conflicts to avoid: -->
<!-- Ctrl+S - Save -->
<!-- Ctrl+C - Copy -->
<!-- Ctrl+V - Paste -->
<!-- Ctrl+Z - Undo -->
<!-- Ctrl+F - Find -->
```

### 2. Use Consistent Patterns

Choose shortcuts that are:

- Easy to remember
- Consistent with common conventions
- Not accidentally triggered

### 3. Document for Users

Create a reference document for your team:

| Shortcut       | Action             |
| -------------- | ------------------ |
| `Ctrl+K`       | Open ProActions    |
| `Ctrl+Shift+K` | Quick improve text |

### 4. Platform Considerations

Remember platform differences:

- Windows/Linux use `Ctrl`
- Mac uses `Cmd`
- Configure both when possible:

```javascript
openHotkey: 'cmd+k,ctrl+k'
```

## Testing Keyboard Shortcuts

### 1. Test in Editor Context

1. Open a document in Swing editor
2. Press your configured shortcut
3. Verify ProActions command palette opens

### 2. Test in Dashboard

1. Navigate to Swing dashboard
2. Press the fallback shortcut
3. Verify ProActions opens

### 3. Test Instant Actions

1. Place cursor in configured context (e.g., body paragraph)
2. Press instant action shortcut
3. Verify action executes automatically

## Troubleshooting Shortcuts

### Shortcut doesn't work in editor

**Check:**

- `/SysConfig/XsmileKbd.cfg` is configured correctly
- XML syntax is valid
- Key character code is correct
- No conflicting shortcuts exist

**Solutions:**

- Reload editor context
- Clear browser cache
- Check browser console for errors
- Try a different key combination

### Shortcut doesn't work in dashboard

**Check:**

- `pro-actions-config.js` has `openHotkey` configured
- Syntax follows hotkeys-js format
- No browser extension conflicts

**Solutions:**

- Verify config file is loaded
- Check browser console for errors
- Try incognito/private mode
- Disable browser extensions

### Instant action doesn't execute

**Check:**

- `instantActionContext` XPath is correct
- Cursor is in the specified context
- Instant action shortcut is configured

**Solutions:**

- Verify XPath matches your document structure
- Use browser inspector to check current element
- Test with simpler XPath first

### Shortcut triggers wrong action

**Check:**

- Only one action has the specific `instantActionContext`
- No duplicate shortcut registrations

**Solutions:**

- Review all instant action configurations
- Use more specific XPath expressions
- Disable conflicting actions temporarily

## Advanced: Custom Shortcut Registration

For advanced use cases, you can programmatically register shortcuts using the Swing API:

```javascript
// Custom shortcut registration - ~/methode-servlets/conf/swing/com.eidosmedia.swing.web-app/app/plugins/pro-actions-custom/my-proactions-shortcuts.js
eidosmedia.webclient.commands.add({
  name: "custom.myProAction",
  label: "My ProAction",
  isActive: function(ctx) { return true; },
  isEnabled: function(ctx) { return true; },
  action: function(ctx, params, callbacks) {
    SwingProActions.executeAction(ctx, "my_action_id");
  }
});
```

```xml
<!-- /SysConfig/XsmileKbd.cfg -->
<context type="Text">
  <key description="K" charCode="80">
    <modifier type="C" shortcut="Ctrl+P">
      <function name="editorExtension" command="custom.myProAction"/>
    </modifier>
  </key>
</context>
```

This allows for:

- Dynamic shortcut registration
- Direct access to ProActions' actions
- Context-aware shortcuts
- Integration with other Swing features

## Example Configurations

### Minimal Setup

```xml
<!-- /SysConfig/XsmileKbd.cfg -->
<context type="Text">
  <key description="K" charCode="75">
    <modifier type="C" shortcut="Ctrl+K">
      <function name="editorExtension" command="custom.openSwingProActions"/>
    </modifier>
  </key>
</context>
```

### Full Setup with Instant Actions

```xml
<!-- /SysConfig/XsmileKbd.cfg -->
<context type="Text">
  <key description="K" charCode="75">
    <modifier type="C" shortcut="Ctrl+K">
      <function name="editorExtension" command="custom.openSwingProActions"/>
    </modifier>
    <modifier type="CS" shortcut="Ctrl+Shift+K">
      <function name="editorExtension" command="custom.instantSwingProActions"/>
    </modifier>
  </key>
</context>
```

```javascript
// pro-actions-config.js
(() => {
  window.SwingProActionsConfig = {
    openHotkey: 'cmd+k,ctrl+k',
    // ... other config ...
  };
})();
```

## Next Steps

- **[Create Your First Flow](../getting-started/your-first-flow.md)** - Build your first workflow
- **[Configuration Guide](./configuration.md)** - Learn about action configuration
- **[Best Practices](../guides/configuration/workflow-design.md)** - Optimize your workflows
