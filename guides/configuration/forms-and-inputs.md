# Forms and Inputs

ProActions provides powerful form-building capabilities to collect user input and display information. This guide explains how to create interactive forms using the FORM step.

## FORM Step Overview

The `FORM` step displays a modal dialog with configurable form fields, allowing users to:

- Enter text, numbers, dates, and other data
- Select from options (dropdowns, radio buttons, checkboxes)
- View formatted content (HTML, Markdown, diffs)
- Choose between multiple actions via custom buttons

## Basic Form Structure

```yaml
- step: FORM
  form:
    # Form field definitions
    fieldName:
      type: text
      label: "Field Label"
      required: true

    # More fields...

  # Button configuration
  buttons:
    - type: cancel
      label: "Cancel"
    - type: submit
      label: "Submit"
```

## Form Field Types

### Text Input

```yaml
userName:
  type: text
  label: "User Name"
  placeholder: "Enter your name"
  required: true
  requiredMessage: "Name is required"
```

### Password Input

```yaml
apiKey:
  type: password
  label: "API Key"
  placeholder: "Enter API key"
  required: true
```

### Textarea

```yaml
comments:
  type: textarea
  label: "Comments"
  placeholder: "Enter your comments"
  rows: 5
  resize: vertical
```

### Number Input

```yaml
wordCount:
  type: number
  label: "Word Count"
  min: 100
  max: 1000
  step: 50
  default: 500
```

The `min` and `max` properties enforce validation constraints. You can also provide custom error messages:

```yaml
age:
  type: number
  label: "Age"
  min: 18
  max: 120
  minMessage: "You must be at least 18 years old"
  maxMessage: "Please enter a valid age (maximum 120)"
  required: true
```

### Select Dropdown

```yaml
language:
  type: select
  label: "Target Language"
  options:
    - English
    - French
    - German
    - Spanish
  default: English
```

### Select with Key-Value Pairs

```yaml
language:
  type: select
  label: "Target Language"
  options:
    - key: EN
      value: English
    - key: FR
      value: French
    - key: DE
      value: German
  default: EN
```

### Radio Buttons

```yaml
tone:
  type: radio
  label: "Article Tone"
  options:
    - Formal
    - Casual
    - Technical
  default: Formal
```

### Checkbox

```yaml
acceptTerms:
  type: checkbox
  label: "I accept the terms and conditions"
  default: false
  required: true
```

### Date Input

```yaml
publishDate:
  type: date
  label: "Publish Date"
  default: "2024-01-01"
```

You can restrict the date range using `min` and `max` properties:

```yaml
appointmentDate:
  type: date
  label: "Appointment Date"
  min: "2025-01-01"
  max: "2025-12-31"
  minMessage: "Appointments start from January 1st, 2025"
  maxMessage: "Appointments must be within 2025"
  required: true
```

### DateTime Input

```yaml
publishDateTime:
  type: datetime
  label: "Publish Date and Time"
```

Similar to date inputs, you can constrain datetime values with `min` and `max`:

```yaml
eventTime:
  type: datetime
  label: "Event Start Time"
  min: "2025-10-24T09:00"
  max: "2025-10-24T18:00"
  minMessage: "Event must start after 9:00 AM"
  maxMessage: "Event must start before 6:00 PM"
  required: true
```

### Color Picker

```yaml
highlightColor:
  type: color
  label: "Highlight Color"
  default: "#FF0000"
```

### Range Slider

```yaml
creativity:
  type: range
  label: "Creativity Level"
  min: 0
  max: 100
  step: 10
  default: 70
```

### File Upload

```yaml
attachment:
  type: file
  label: "Upload File"
  accept: "image/*,.pdf"
  maxSize: 5242880  # 5MB in bytes
  maxSizeMessage: "File must be less than 5MB"
```

## Display Components

### Headline

```yaml
sectionTitle:
  type: headline
  text: "Section Title"
```

### Horizontal Rule

```yaml
separator:
  type: hr
```

### HTML Content

```yaml
instructions:
  type: html
  html: |
    <p><strong>Instructions:</strong></p>
    <ul>
      <li>Fill in all required fields</li>
      <li>Review your input before submitting</li>
    </ul>
```

### Markdown Content

```yaml
description:
  type: markdown
  text: |
    ## Important Information

    Please read carefully:
    - **Bold text**
    - *Italic text*
    - [Links](https://example.com)
```

### Diff Display

Show differences between two texts:

```yaml
comparison:
  type: diff
  label: "Changes"
  prevText: "{{ flowContext.originalText }}"
  text: "{{ flowContext.newText }}"
  mode: words  # Options: chars, words, lines, sentences
  showDeletions: true
  interactive: false
```

### Diff Toolbar Controls

Customize toolbar behavior for interactive diff review:

```yaml
comparison:
  type: diff
  prevText: "{{ flowContext.originalText }}"
  text: "{{ flowContext.newText }}"
  interactive: true
  diffControls:
    position: top            # top | bottom
    zoom: true
    acceptRejectAll: true
    acceptRejectAllEnabled: true
    showStats: true          # shows Length: source → target
    symbols:
      zoomIn: "A+"
      zoomOut: "A−"
      acceptAll: "✔"
      rejectAll: "✖"
```

When `showStats` is enabled, the indicator displays `Length: <source> → <target>`. In interactive mode, the target value updates live as additions/deletions/replacements are selected.

## Advanced Form Features

### Field Validation

```yaml
email:
  type: text
  label: "Email Address"
  required: true
  validation:
    pattern: "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$"
    errorMessage: "Please enter a valid email address"
```

### Conditional Visibility

Show fields based on other field values:

```yaml
translationService:
  type: select
  label: "Translation Service"
  options:
    - DeepL
    - Google Translate

deeplApiKey:
  type: password
  label: "DeepL API Key"
  visibleIf:
    translationService: DeepL
```

### Tooltips

```yaml
temperature:
  type: range
  label: "Temperature"
  min: 0
  max: 1
  step: 0.1
  default: 0.7
  tooltip: "Higher values make output more creative, lower values more deterministic"
```

## Form Buttons

### Submit Button

```yaml
buttons:
  - type: submit
    label: "Submit"
```

### Cancel Button

```yaml
buttons:
  - type: cancel
    label: "Cancel"
```

### Multiple Submit Buttons

Create different actions based on which button is clicked:

```yaml
buttons:
  - type: cancel
    label: "Discard"

  - type: submit
    flowAttribute: "action"
    name: "copy"
    class: "btn btn-info"
    label: "Copy to Clipboard"

  - type: submit
    flowAttribute: "action"
    name: "replace"
    label: "Replace Text"
```

Then handle the action:

```yaml
- step: SWITCH
  condition: "{{ flowContext.action }}"
  cases:
    'copy':
      - step: WRITE_CLIPBOARD
    'replace':
      - step: REPLACE_TEXT
        at: CURSOR
```

## Modal Configuration

Configure the modal dialog appearance using the `formConfig` option:

```yaml
- step: FORM
  title: "My Form"
  formConfig:
    dialogSize: lg  # Options: sm, md, lg, xl
    width: "800px"
    maxWidth: "90vw"
    height: "600px"
    maxHeight: "80vh"
    fullScreen: false
  form:
    # fields...
```

### Modal Size Options

**Predefined Sizes:**

```yaml
formConfig:
  dialogSize: sm    # Small modal (~300px)
  dialogSize: md    # Medium (default, ~600px)
  dialogSize: lg    # Large (~900px)
  dialogSize: xl    # Extra large (~1200px or 90vw)
```

**Custom Dimensions:**

```yaml
formConfig:
  # Width options
  width: "720px"      # Fixed width in pixels
  width: "80vw"       # 80% of viewport width
  maxWidth: "1200px"  # Maximum width constraint

  # Height options
  height: "500px"     # Fixed height in pixels
  height: "70vh"      # 70% of viewport height
  maxHeight: "90vh"   # Maximum height constraint

  # Fullscreen mode
  fullScreen: true    # Cover entire viewport (100vw x 100vh)
```

**Important Notes:**

- When `height` or `maxHeight` is set, the modal body becomes scrollable
- The header and footer remain fixed while the body scrolls
- Fullscreen mode overrides width and height settings
- The modal auto-adjusts if requested height exceeds viewport

### Modal Typography

Customize text appearance throughout the modal:

```yaml
formConfig:
  # Modal body styling
  bodyFontSize: "16px"
  bodyLineHeight: "1.6"

  # Form labels
  labelFontSize: "14px"

  # Input fields (text, textarea, select, etc.)
  inputFontSize: "15px"
  inputLineHeight: "1.5"

  # Diff component specific
  diffFontSize: "13px"
  diffLineHeight: "1.4"
```

**Supported Units:**

- Pixels: `"14px"`, `"16px"`
- Relative: `"1rem"`, `"1.2em"`
- Line height: `"1.5"`, `"1.6"`

### Custom CSS Class

Add custom CSS classes for additional styling:

```yaml
formConfig:
  modalClass: "my-custom-modal"
```

Then define styles in your custom CSS:

```css
.my-custom-modal .modal-dialog {
  border-radius: 8px;
  box-shadow: 0 4px 20px rgba(0,0,0,0.2);
}
```

### Complete formConfig Reference

| Option            | Type    | Description                             | Example              |
| ----------------- | ------- | --------------------------------------- | -------------------- |
| `dialogSize`      | string  | Predefined size: 'sm', 'md', 'lg', 'xl' | `'lg'`               |
| `width`           | string  | Custom width (px or vw)                 | `'80vw'`, `'1000px'` |
| `maxWidth`        | string  | Maximum width                           | `'1200px'`           |
| `height`          | string  | Custom height (px or vh)                | `'70vh'`, `'600px'`  |
| `maxHeight`       | string  | Maximum height                          | `'800px'`            |
| `fullScreen`      | boolean | Cover entire viewport                   | `true`               |
| `bodyFontSize`    | string  | Modal body font size                    | `'16px'`, `'1rem'`   |
| `bodyLineHeight`  | string  | Modal body line height                  | `'1.6'`              |
| `labelFontSize`   | string  | Form label font size                    | `'14px'`             |
| `inputFontSize`   | string  | Input field font size                   | `'15px'`             |
| `inputLineHeight` | string  | Input field line height                 | `'1.5'`              |
| `diffFontSize`    | string  | Diff component font size                | `'13px'`             |
| `diffLineHeight`  | string  | Diff component line height              | `'1.4'`              |
| `modalClass`      | string  | Custom CSS class name                   | `'my-modal-class'`   |

### Modal Configuration Examples

**Example 1: Large Form with Scrolling**

```yaml
- step: FORM
  title: "Content Editor"
  formConfig:
    dialogSize: 'xl'
    height: '85vh'
    maxHeight: '900px'
  form:
    content:
      type: textarea
      rows: 20
      resize: vertical
    metadata:
      type: text
      label: "Metadata"
```

**Example 2: Fullscreen Review Interface**

```yaml
- step: FORM
  title: "Review Changes"
  formConfig:
    fullScreen: true
    diffFontSize: '14px'
    diffLineHeight: '1.5'
  form:
    comparison:
      type: diff
      mode: words
      prevText: '{{ flowContext.original }}'
      text: '{{ flowContext.improved }}'
      interactive: true
```

**Example 3: Compact Modal with Custom Typography**

```yaml
- step: FORM
  title: "Quick Settings"
  formConfig:
    dialogSize: 'sm'
    bodyFontSize: '13px'
    labelFontSize: '12px'
    inputFontSize: '13px'
  form:
    option1:
      type: checkbox
      label: "Enable feature A"
    option2:
      type: checkbox
      label: "Enable feature B"
```

**Example 4: Responsive Modal**

```yaml
- step: FORM
  title: "Translation Settings"
  formConfig:
    width: '90vw'
    maxWidth: '1000px'
    height: '80vh'
    maxHeight: '700px'
    inputFontSize: '14px'
  form:
    sourceText:
      type: textarea
      label: "Source Text"
      rows: 8
    targetLang:
      type: select
      label: "Target Language"
      options:
        - English
        - French
        - German
```

## Complete Form Examples

### Example 1: Simple Input Form

```yaml
- title: 'Generate Custom Content'
  flow:
    - step: FORM
      form:
        topic:
          type: text
          label: "Topic"
          placeholder: "Enter article topic"
          required: true

        wordCount:
          type: number
          label: "Word Count"
          min: 100
          max: 2000
          default: 500

        tone:
          type: select
          label: "Tone"
          options:
            - Professional
            - Casual
            - Academic

      buttons:
        - type: cancel
        - type: submit
          label: "Generate"

    - step: HUB_COMPLETION
      instruction: |
        Write a {{ flowContext.wordCount }}-word article about {{ flowContext.topic }}
        in a {{ flowContext.tone }} tone.

    - step: INSERT_TEXT
      at: CURSOR
```

### Example 2: Translation Form

```yaml
- title: 'Configure Translation'
  flow:
    - step: FORM
      form:
        headline1:
          type: headline
          text: "Translation Settings"

        targetLang:
          type: select
          label: "Target Language"
          options:
            - key: FR
              value: French
            - key: DE
              value: German
            - key: ES
              value: Spanish
          required: true

        formality:
          type: radio
          label: "Formality"
          options:
            - default
            - more
            - less
          default: default

        preserveFormatting:
          type: checkbox
          label: "Preserve formatting"
          default: true

      buttons:
        - type: cancel
        - type: submit
          label: "Translate"

    - step: DEEPL_TRANSLATE
      instruction: "{selectedText}"
      target_lang: "{{ flowContext.targetLang }}"
      formality: "{{ flowContext.formality }}"

    - step: REPLACE_TEXT
      at: CURSOR
```

### Example 3: Diff Display Form

```yaml
- title: 'Review Text Improvements'
  flow:
    - step: SET
      currentText: "{selectedText}"

    - step: DEEPL_WRITE
      instruction: "{{ flowContext.currentText | safe }}"

    - step: FORM
      modal:
        width: "900px"
        height: "700px"
      form:
        headline1:
          type: headline
          text: "Current Text"

        original:
          type: html
          html: "<p><i>{{ flowContext.currentText }}</i></p>"

        separator:
          type: hr

        headline2:
          type: headline
          text: "Improved Text"

        comparison:
          type: diff
          mode: words
          prevText: "{{ flowContext.currentText }}"
          text: "{{ flowContext.text }}"
          showDeletions: false

      buttons:
        - type: cancel
          label: "Discard"
        - type: submit
          flowAttribute: "mode"
          name: "copy"
          class: "btn btn-info"
          label: "Copy"
        - type: submit
          flowAttribute: "mode"
          name: "replace"
          label: "Replace"

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
            message: "Copied to clipboard"
```

### Example 4: File Upload Form

```yaml
- title: 'Upload and Process Image'
  flow:
    - step: FORM
      form:
        instructions:
          type: html
          html: |
            <p>Upload an image to process</p>

        imageFile:
          type: file
          label: "Select Image"
          accept: "image/*"
          maxSize: 10485760  # 10MB
          required: true

        operation:
          type: select
          label: "Operation"
          options:
            - Upscale
            - Anonymize
            - Enhance

        buttons:
          - type: cancel
          - type: submit
            label: "Process"

    # Process based on selected operation
    - step: SWITCH
      condition: "{{ flowContext.operation }}"
      cases:
        'Upscale':
          - step: STABILITY_AI_UPSCALE
        'Anonymize':
          - step: BRIGHTER_AI
```

## User Input Steps

In addition to FORM, ProActions provides simpler input steps:

### PROMPT Step

Simple text input:

```yaml
- step: PROMPT
  promptText: "Enter your search query:"

- step: SHOW_NOTIFICATION
  message: "You entered: {{ flowContext.text }}"
```

### USER_SELECT Step

Choose from a list:

```yaml
- step: HUB_COMPLETION
  instruction: "Generate 5 headline options"
  response_format: "list"

- step: USER_SELECT
  promptText: "Choose a headline:"

- step: REPLACE_TEXT
  at: XPATH
  xpath: "/doc/story/headline/p"
```

### FILE_UPLOAD Step

Upload files:

```yaml
- step: FILE_UPLOAD
  accept: "audio/*"
  multiple: false

- step: HUB_TRANSCRIPTION
```

## Form Best Practices

### 1. Provide Clear Labels

```yaml
# Good
targetLanguage:
  type: select
  label: "Target Language for Translation"
  tooltip: "Select the language you want to translate to"

# Avoid
lang:
  type: select
  label: "Lang"
```

### 2. Use Appropriate Input Types

```yaml
# Good - number type with constraints
wordCount:
  type: number
  min: 100
  max: 5000

# Avoid - text type for numbers
wordCount:
  type: text
  placeholder: "Enter 100-5000"
```

### 3. Set Sensible Defaults

```yaml
temperature:
  type: range
  label: "Creativity"
  min: 0
  max: 1
  step: 0.1
  default: 0.7  # Sensible default
```

### 4. Validate User Input

```yaml
url:
  type: text
  label: "Website URL"
  required: true
  validation:
    pattern: "^https?://.+"
    errorMessage: "Please enter a valid URL starting with http:// or https://"
```

### 5. Group Related Fields

```yaml
form:
  headline1:
    type: headline
    text: "Content Settings"

  topic:
    type: text
    label: "Topic"

  wordCount:
    type: number
    label: "Word Count"

  separator:
    type: hr

  headline2:
    type: headline
    text: "Style Settings"

  tone:
    type: select
    label: "Tone"
```

### 6. Provide Helpful Instructions

```yaml
instructions:
  type: html
  html: |
    <p><strong>Please configure your translation:</strong></p>
    <ul>
      <li>Select target language</li>
      <li>Choose formality level</li>
      <li>Review before submitting</li>
    </ul>
```

## Next Steps

- **[Configuration Basics](./basics.md)** - Overall configuration concepts
- **[Menus and Triggers](./menus-and-triggers.md)** - Configure menus and shortcuts
