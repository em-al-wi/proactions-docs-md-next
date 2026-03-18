# DeepL Write Integration

Improve text quality using DeepL Write with before/after comparison and flexible actions.

## Overview

This example demonstrates how to:

- Use the DEEPL_WRITE step for AI-powered text improvement
- Display before/after comparison using diff components in forms
- Implement custom form buttons with different actions
- Write content to clipboard (WRITE_CLIPBOARD)
- Use SWITCH step for conditional logic based on button selection
- Configure Swing's inline toolbar integration for contextual access

## Requirements

- DeepL API Pro Account with DeepL WRITE access
- ProActions Hub configured as proxy endpoint
- Browser with clipboard access

## Use Case

DeepL Write provides professional-grade text improvement for:

- Grammar and spelling correction
- Style enhancement
- Clarity improvements
- Tone adjustments

This workflow integrates DeepL Write seamlessly into the editorial process with options to replace text or copy to clipboard.

## Configuration

### Action Definition

```yaml
# pro-actions.ai-kit.yaml
AI_KIT:
  SERVICES:
    # Use ProActions Hub as endpoint for DeepL Write
    DEEPL_WRITE:
      endpoint: /swing/proactions/proxy/forward/deepl-write

  SECTIONS:
    - section: Examples
      actions:
        # Configure the action to send text to DeepL Write
        - title: 'Send selected text to DeepL Write'
          id: deeplWriteSelection
          swing:
            inlineMenu:
              enable: true
              icon: "fa fa-pencil"
              label: "  DeepL Write"
              allowReadOnly: false
          flow:
            # Store current text
            - step: SET
              currentText: "{selectedText}"

            # Send to DeepL Write
            - step: DEEPL_WRITE
              instruction: "{{ flowContext.currentText | safe }}"

            # Show comparison in form
            - step: FORM
              form:
                # Current text section
                headline1:
                  type: "headline"
                  text: "Current Text"

                inlineHtml:
                  type: "html"
                  html: |
                    <p><i>{{ flowContext.currentText }}</i></p>

                hrNew:
                  type: "hr"

                # Improved text section with DeepL logo
                headlineNew:
                  type: "html"
                  html: |
                    <p><svg aria-label="DeepL" width="100" viewBox="0 0 545 184" fill="none" xmlns="http://www.w3.org/2000/svg">
                      <!-- DeepL logo SVG paths -->
                    </svg></p>

                # Diff component showing changes
                inlineDiff:
                  type: "diff"
                  mode: "words"
                  prevText: "{{ flowContext.currentText }}"
                  text: "{{ flowContext.text }}"
                  showDeletions: false

              # Custom buttons
              buttons:
                - type: "cancel"
                  label: "Discard"

                - type: "submit"
                  flowAttribute: "mode"
                  name: "copy"
                  class: "btn btn-info"
                  label: "Copy"

                - type: "submit"
                  flowAttribute: "mode"
                  name: "replace"
                  label: 'Replace'

            # Handle button action
            - step: SWITCH
              condition: "{{ flowContext.mode }}"
              cases:
                'replace':
                  - step: REPLACE_TEXT
                    at: CURSOR

                'copy':
                  - step: WRITE_CLIPBOARD

                  - step: SHOW_NOTIFICATION
                    notificationType: success
                    message: "Copied text to clipboard"
                    title: "ProActions"
```

## How It Works

### Step 1: Capture Selected Text

Store the original text for comparison:

```yaml
- step: SET
  currentText: "{selectedText}"
```

This preserves the original so it can be shown alongside the improved version.

### Step 2: DeepL Write Processing

Send text to DeepL Write for improvement:

```yaml
- step: DEEPL_WRITE
  instruction: "{{ flowContext.currentText | safe }}"
```

**Key points:**

- Uses the `| safe` filter to prevent XML escaping
- Sends request through ProActions Hub proxy
- Returns improved text in `flowContext.text`

### Step 3: Form with Diff Component

Display before/after comparison:

```yaml
- step: FORM
  form:
    inlineDiff:
      type: "diff"
      mode: "words"
      prevText: "{{ flowContext.currentText }}"
      text: "{{ flowContext.text }}"
      showDeletions: false
```

**Diff component options:**

- `mode: "words"` - Highlight word-level changes (also: `"chars"`, `"lines"`)
- `prevText` - Original text
- `text` - New text
- `showDeletions: false` - Hide deleted text, only show additions

### Step 4: Custom Form Buttons

Define multiple action buttons:

```yaml
buttons:
  - type: "cancel"
    label: "Discard"

  - type: "submit"
    flowAttribute: "mode"  # Stores button value in flowContext.mode
    name: "copy"
    class: "btn btn-info"
    label: "Copy"

  - type: "submit"
    flowAttribute: "mode"
    name: "replace"
    label: 'Replace'
```

**Button types:**

- `cancel` - Closes form without continuing flow
- `submit` - Submits form and continues flow
- `flowAttribute` - Variable name to store button's `name` value

### Step 5: Conditional Actions with SWITCH

Execute different actions based on button pressed:

```yaml
- step: SWITCH
  condition: "{{ flowContext.mode }}"
  cases:
    'replace':
      - step: REPLACE_TEXT
        at: CURSOR

    'copy':
      - step: WRITE_CLIPBOARD
      - step: SHOW_NOTIFICATION
        message: "Copied text to clipboard"
        type: success
```

The SWITCH step checks `flowContext.mode` (set by the button) and executes the corresponding case.

## Swing Inline Toolbar Integration

This example uses Swing's native inline toolbar integration to add a DeepL Write button that appears when selecting text in the editor.

For complete documentation on Swing's `inlineMenu` integration, including:
- All configuration options
- Conditional visibility expressions
- Best practices and troubleshooting
- Migration from deprecated TEXT_MENU

See the **[Swing Integration Guide](../configuration/swing-integration.md#inline-toolbar-integration)**.

## Best Practices

1. **Preserve Original**: Always store original text for comparison
2. **Show Changes**: Use diff component to highlight improvements
3. **Multiple Options**: Provide copy and replace options for flexibility
4. **User Control**: Let users review before accepting changes
5. **Hub Proxy**: Use ProActions Hub to secure API keys

## Troubleshooting

**Diff not showing changes:**

- Verify both `prevText` and `text` are populated
- Check that text actually changed
- Try different `mode` (words, chars, lines)
- Ensure no HTML escaping issues

**Clipboard write fails:**

- Check browser clipboard permissions
- Some browsers require user gesture (button click)
- Ensure content isn't empty
- Try in HTTPS context

## Related Examples

- [Headline Suggestions](./headline-suggestions.md) - Generate headline options
- [Progress Bar](./progress-bar.md) - Show progress for batch operations
