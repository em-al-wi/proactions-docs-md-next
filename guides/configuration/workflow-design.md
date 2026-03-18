# Workflow Design Best Practices

This guide covers best practices for designing effective, maintainable, and user-friendly ProActions workflows.

## Design Principles

### 1. Keep It Simple

Start with the simplest solution that solves the problem.

**Good - Simple and clear:**

```yaml
- title: 'Improve Text'
  flow:
    - step: HUB_COMPLETION
      instruction: 'Improve this text: {selectedText}'
    - step: REPLACE_TEXT
      at: CURSOR
```

**Avoid - Unnecessarily complex:**

```yaml
- title: 'Improve Text'
  flow:
    - step: GET_TEXT_CONTENT
      at: CURSOR
    - step: SET
      originalText: '{{ flowContext.selectedText }}'
    - step: IF
      condition: '{{ flowContext.originalText }}'
      then:
        - step: HUB_COMPLETION
          instruction: 'Improve: {{ flowContext.originalText }}'
        - step: SET
          improvedText: '{{ flowContext.text }}'
        - step: REPLACE_TEXT
          at: CURSOR
```

### 2. Single Responsibility

Each action should do one thing well.

**Good - Focused actions:**

```yaml
- title: 'Translate to French'
  flow:
    - step: DEEPL_TRANSLATE
      instruction: '{selectedText}'
      target_lang: 'FR'
    - step: REPLACE_TEXT
      at: CURSOR

- title: 'Translate to German'
  flow:
    - step: DEEPL_TRANSLATE
      instruction: '{selectedText}'
      target_lang: 'DE'
    - step: REPLACE_TEXT
      at: CURSOR
```

**Avoid - Multi-purpose actions:**

```yaml
- title: 'Translate'
  flow:
    - step: PROMPT
      promptText: 'Choose language: FR, DE, ES'
    - step: DEEPL_TRANSLATE
      instruction: '{selectedText}'
      target_lang: '{{ flowContext.text }}'
    # Complex, error-prone, harder to use
```

### 3. Progressive Disclosure

Show complexity only when needed.

**Good - Start simple, add options:**

```yaml
# Basic action
- title: 'Generate Headline'
  flow:
    - step: HUB_COMPLETION
      instruction: 'Generate headline for: {textContent}'
    - step: REPLACE_TEXT
      at: XPATH
      xpath: '/doc/story/headline/p'

# Advanced action with options
- title: 'Generate Headline (Advanced)'
  flow:
    - step: FORM
      form:
        count:
          type: number
          label: 'Number of options'
        tone:
          type: select
          label: 'Tone'
          options: [Professional, Casual, Urgent]
    - step: HUB_COMPLETION
      instruction: 'Generate {{ flowContext.count }} {{ flowContext.tone }} headlines'
      response_format: 'list'
    - step: USER_SELECT
      promptText: 'Choose headline:'
    - step: REPLACE_TEXT
      at: XPATH
      xpath: '/doc/story/headline/p'
```

## Naming Conventions

### Action Titles

Use clear, action-oriented titles:

**Good:**

```yaml
- title: 'Translate to French'
- title: 'Generate Headline Suggestions'
- title: 'Improve Selected Text'
- title: 'Create Audio Version'
```

**Avoid:**

```yaml
- title: 'French' # Unclear what it does
- title: 'Headlines' # Is it generating or editing?
- title: 'AI' # Too vague
- title: 'Process' # What kind of processing?
```

### Section Names

Group related actions under clear section names:

**Good:**

```yaml
section: Translation Tools
section: Content Generation
section: Text Enhancement
section: Image Processing
```

**Avoid:**

```yaml
section: AI Tools  # Too broad
section: Stuff  # Unprofessional
section: My Actions  # Not descriptive
```

### Variable Names

Use descriptive variable names:

**Good:**

```yaml
- step: SET
  articleTitle: '{{ client.getMetadata().metadata.general.title }}'
  targetLanguage: 'FR'
  wordCountLimit: 500
```

**Avoid:**

```yaml
- step: SET
  t: '{{ client.getMetadata().metadata.general.title }}'
  lang: 'FR'
  n: 500
```

## User Experience

### Provide Feedback

Always inform users about what's happening:

**Good - With feedback:**

```yaml
- title: 'Generate Article'
  flow:
    - step: SHOW_PROGRESS
      status: 'Generating article...'

    - step: HUB_COMPLETION
      instruction: 'Write article about: {selectedText}'

    - step: UPDATE_PROGRESS
      percentage: 50
      status: 'Doing something else'

    - step: HUB_COMPLETION
      instruction: 'Do something else: {selectedText}'

    - step: HIDE_PROGRESS

    - step: SHOW_NOTIFICATION
      notificationType: success
      message: 'Article generated successfully'

    - step: INSERT_TEXT
      at: CURSOR
```

**Avoid - No feedback:**

```yaml
- title: 'Generate Article'
  flow:
    - step: HUB_COMPLETION
      instruction: 'Write article about: {selectedText}'

    - step: INSERT_TEXT
      at: CURSOR
```

### Use Background Processing (inlineSteps)

For UI steps that require data preparation, use `inlineSteps` to show the interface immediately while processing happens in the background. This improves perceived performance by eliminating wait time.

**Supported Steps:** `USER_SELECT`, `FORM`, `SHOW_RESPONSE`

**Good - Immediate UI with background processing:**

```yaml
- title: 'Suggest Headlines'
  flow:
    - step: USER_SELECT
      inlineSteps:
        - step: HUB_COMPLETION
          instruction: 'Generate 6 headlines for: {textContent}'
          response_format: 'list'
      promptText: 'Select a headline'
      enableKeyboardControl: true

    - step: REPLACE_TEXT
      at: XPATH
      xpath: '/doc/story/headline/p'
```

**How it works:**

- The modal displays immediately with a loading indicator
- `inlineSteps` execute in the background
- When processing completes, results populate the UI seamlessly
- Errors are displayed in-place within the modal

### Validate Input

Check for required input before processing:

**Good:**

```yaml
- title: 'Translate Selection'
  flow:
    - step: IF
      condition: '{{ client.isTextSelected() }}'
      then:
        - step: DEEPL_TRANSLATE
          instruction: '{selectedText}'
          target_lang: 'FR'
        - step: REPLACE_TEXT
          at: CURSOR
      else:
        - step: SHOW_NOTIFICATION
          notificationType: warning
          message: 'Please select text to translate'
```

**Avoid:**

```yaml
- title: 'Translate Selection'
  flow:
    - step: DEEPL_TRANSLATE
      instruction: '{selectedText}'
      # Fails if no text selected
    - step: REPLACE_TEXT
      at: CURSOR
```

### Offer Choices

Let users make decisions when appropriate:

**Good:**

```yaml
- title: 'Generate Headlines'
  flow:
    - step: HUB_COMPLETION
      instruction: 'Generate 5 headline options'
      response_format: 'list'

    - step: USER_SELECT
      promptText: 'Choose a headline:'

    - step: REPLACE_TEXT
      at: XPATH
      xpath: '/doc/story/headline/p'
```

### Provide Escape Routes

Always give users a way to cancel:

**Good:**

```yaml
- step: FORM
  form:
    topic:
      type: text
      label: 'Article Topic'
      required: true

  buttons:
    - type: cancel
      label: 'Cancel'
    - type: submit
      label: 'Generate'
```

### Graceful Flow Termination

Use the `END` step with notifications to terminate workflows gracefully while informing users:

**Good - End with notification:**

```yaml
- title: 'Process Article'
  flow:
    - step: END
      condition: '{{ not client.isTextSelected() }}'
      notification:
        type: 'warning'
        message: 'Please select text before running this action'
        title: 'No Selection'

    - step: HUB_COMPLETION
      instruction: 'Process: {selectedText}'

    - step: REPLACE_TEXT
      at: CURSOR
```

**Also good - Conditional end with context:**

```yaml
- title: 'Smart Translation'
  flow:
    - step: SET
      wordCount: "{{ client.getTextContent().split(' ').length }}"
      convertTo: number

    - step: END
      condition: '{{ flowContext.wordCount > 10000 }}'
      notification:
        type: 'error'
        message: 'Text is too long ({{flowContext.wordCount}} words). Maximum is 10,000 words.'
        title: 'Text Too Long'

    - step: DEEPL_TRANSLATE
      instruction: '{textContent}'
      target_lang: 'FR'
```

**Avoid - Silent termination:**

```yaml
- step: END
  condition: '{{ someCondition }}'
  # User doesn't know why workflow stopped
```

## Error Handling

### Use TRY/Catch

Always handle potential errors:

**Good:**

```yaml
- title: 'Process with AI'
  flow:
    - step: TRY
      try:
        - step: HUB_COMPLETION
          instruction: 'Process: {textContent}'

        - step: SHOW_NOTIFICATION
          notificationType: success
          message: 'Processing complete'
      catch:
        - step: SHOW_NOTIFICATION
          notificationType: error
          message: 'Processing failed. Please try again.'
```

### Graceful Degradation

Provide alternatives when services fail:

**Good:**

```yaml
- title: 'Smart Translation'
  flow:
    - step: TRY
      try:
        - step: DEEPL_TRANSLATE
          instruction: '{selectedText}'
          target_lang: 'FR'
        - step: REPLACE_TEXT
          at: CURSOR
      catch:
        - step: SHOW_NOTIFICATION
          notificationType: warning
          message: 'Translation service unavailable. Using fallback...'

        - step: HUB_COMPLETION
          instruction: 'Translate to French: {selectedText}'

        - step: REPLACE_TEXT
          at: CURSOR
```

### Action-Level Error Handling

For simpler error handling scenarios, use the `onError` configuration at the action level. This provides a way to handle errors for the entire action without wrapping individual steps in `TRY/CATCH` blocks:

```yaml
- title: 'Extract Keywords'
  flow:
    - step: HUB_COMPLETION
      instruction: 'Extract keywords from: {textContent}'
      response_format: 'list'

  onError:
    - step: SHOW_NOTIFICATION
      notificationType: error
      message: 'Failed to extract keywords. Please try again.'
```

**How it works:**

- If any step in the `flow` throws an error, execution stops
- The steps in `onError` are executed instead
- The error is available in `flowContext.error`
- After `onError` steps complete, the error is re-thrown (unless handled)

**When to use `onError` vs `TRY/CATCH`:**

- **Use `onError`** when you need consistent error handling for an entire action
- **Use `TRY/CATCH`** when you need granular error handling for specific steps within a flow
- **Combine both** for complex scenarios with both action-level and step-level error handling

## Organization and Maintainability

### Comment Complex Workflows

Document your intentions:

**Good:**

```yaml
- title: 'Process Article'
  flow:
    # Step 1: Extract article content excluding metadata
    - step: SET
      text: '{{ client.getTextContent({ excludeTags: ["caption", "credit"] }) | safe }}'

    # Step 2: Send to AI for processing
    - step: HUB_COMPLETION
      instruction: 'Summarize: {{ flowContext.text }}'

    # Step 3: Show result to user for approval
    - step: SHOW_RESPONSE
      message: '{{ flowContext.text }}'
```

### Use Consistent Patterns

Establish and follow patterns across your workflows:

**Good - Consistent pattern:**

```yaml
# All translation actions follow same pattern
- title: 'Translate to French'
  flow:
    - step: DEEPL_TRANSLATE
      instruction: '{selectedText}'
      target_lang: 'FR'
    - step: REPLACE_TEXT
      at: CURSOR

- title: 'Translate to German'
  flow:
    - step: DEEPL_TRANSLATE
      instruction: '{selectedText}'
      target_lang: 'DE'
    - step: REPLACE_TEXT
      at: CURSOR
```

## Performance Considerations

### Limit Token Usage

Set reasonable token limits for LLM calls:

**Good:**

```yaml
- step: HUB_COMPLETION
  instruction: 'Generate headline: {textContent}'
  max_tokens: 100 # Headlines don't need many tokens
```

**Avoid:**

```yaml
- step: HUB_COMPLETION
  instruction: 'Generate headline: {textContent}'
  # No limit - may waste tokens
```

### Use Appropriate Response Formats

Request only the format you need:

**Good:**

```yaml
# For lists
- step: HUB_COMPLETION
  instruction: 'Generate 5 topics'
  response_format: 'list'

# For structured data
- step: HUB_COMPLETION
  instruction: 'Analyze sentiment and extract topics'
  response_format: 'json_object'

# For simple text
- step: HUB_COMPLETION
  instruction: 'Improve this text'
  response_format: 'text'
```

### Avoid Unnecessary Steps

Eliminate redundant operations:

**Good:**

```yaml
- title: 'Translate'
  flow:
    - step: DEEPL_TRANSLATE
      instruction: '{selectedText}'
      target_lang: 'FR'
    - step: REPLACE_TEXT
      at: CURSOR
```

**Avoid:**

```yaml
- title: 'Translate'
  flow:
    - step: GET_TEXT_CONTENT
      at: CURSOR
    - step: SET
      textToTranslate: '{{ flowContext.selectedText }}'
    - step: DEEPL_TRANSLATE
      instruction: '{{ flowContext.textToTranslate }}'
    - step: SET
      translatedText: '{{ flowContext.text }}'
    - step: REPLACE_TEXT
      at: CURSOR
```

## Configuration Best Practices

### Set Defaults

Provide sensible defaults:

**Good:**

```yaml
- step: HUB_COMPLETION
  instruction: 'Generate content'
  model: 'gpt-4'
  options:
    temperature: 0.7
    max_tokens: 500
```

### Use Templates for Reusable Logic

Define templates for common operations:

```yaml
# In TEMPLATES section
TEMPLATES:
  improveText:
    - step: HUB_COMPLETION
      behavior: "You are a professional editor."
      instruction: "Improve: {selectedText}"
    - step: REPLACE_TEXT
      at: CURSOR

# In actions
- title: 'Quick Improve'
  flow:
    - step: CALL_TEMPLATE
      name: improveText
```

### Prefer Scoped Reusable Blocks for New Work

For new configurations, use scoped reusable blocks (`BLOCKS`, `section.blocks`, `action.blocks`) and call flows/scripts by reference.

```yaml
AI_KIT:
  BLOCKS:
    flows:
      improveText:
        - step: HUB_COMPLETION
          behavior: 'You are a professional editor.'
          instruction: 'Improve: {selectedText}'
        - step: REPLACE_TEXT
          at: CURSOR

  SECTIONS:
    - section: Editing
      blocks:
        vars:
          improveFlowRef: global.flows.improveText
      actions:
        - title: Quick Improve
          flow:
            - step: CALL_FLOW
              ref: '{{ section.vars.improveFlowRef }}'
```

### Migrate Gradually from `TEMPLATES`

- Keep existing `CALL_TEMPLATE.name` actions as-is.
- Add new reusable logic under `BLOCKS` and start using `CALL_FLOW.ref`.
- If needed, use `CALL_TEMPLATE.ref` as a bridge while migrating.
- For scripts, move inline `SCRIPTING.script` snippets to scoped `scripts` and reference them with `SCRIPTING.scriptRef`.

### Imports Resolution Rules

When using reusable modules via `imports`, keep these runtime rules in mind:

- Section imports are automatically available to every action in the section.
- Action imports are merged with section imports for that action.
- If section and action both define the same alias, the action alias overrides in that action scope.
- Defining the same alias twice in a single `imports` list is invalid and fails at load time.
- Circular import chains are detected and rejected.

```yaml
SECTIONS:
  - section: Editorial
    imports:
      - from: /SysConfig/ProActions/lib/common.ai-kit.module.yaml
        as: common
    actions:
      - title: Editorial Rewrite
        imports:
          - from: /SysConfig/ProActions/lib/editorial.ai-kit.module.yaml
            as: common # overrides section alias for this action only
        flow:
          - step: CALL_FLOW
            ref: imports.common.flows.rewrite
```

### Centralize Service Configuration

Keep service configs in one place:

```yaml
# services.ai-kit.services.yaml
HUB:
  endpoint: '{BASE_URL}/swing/proactions'
  target: openai
  defaultBehavior: |
    You are a professional writing assistant.
    Provide clear, concise responses.

DEEPL:
  endpoint: /swing/proactions/proxy/forward/deepl
```

## Testing and Validation

### Test with Real Users

Get feedback from actual users:

- Is the action name clear?
- Is the workflow intuitive?
- Are error messages helpful?
- Is feedback adequate?

### Version Your Workflows

Document changes and maintain versions:

```yaml
# headlines.yaml
# Version: 2.0
# Last updated: 2024-01-15
# Changes: Added tone selection, improved prompts

section: Headlines
actions:
  - title: 'Generate Headlines'
    # workflow...
```

## Accessibility

### Use Clear Language

Write for all skill levels:

**Good:**

```yaml
- step: SHOW_NOTIFICATION
  message: 'Translation complete. Your text has been replaced.'
```

**Avoid:**

```yaml
- step: SHOW_NOTIFICATION
  message: 'NLP proc complete. Obj updated.'
```

### Provide Help Text

Use tooltips and descriptions:

```yaml
- step: FORM
  form:
    temperature:
      type: range
      label: 'Creativity Level'
      tooltip: 'Higher values produce more creative results, lower values are more predictable'
      min: 0
      max: 1
      step: 0.1
      default: 0.7
```

## Common Patterns

### Pattern: Preview Before Replace

```yaml
- title: 'Improve with Preview'
  flow:
    - step: SET
      originalText: '{selectedText}'

    - step: HUB_COMPLETION
      instruction: 'Improve: {{ flowContext.originalText }}'

    - step: FORM
      form:
        comparison:
          type: diff
          prevText: '{{ flowContext.originalText }}'
          text: '{{ flowContext.text }}'
          showDeletions: true
          interactive: true
      buttons:
        - type: cancel
          label: 'Keep Original'
        - type: submit
          label: 'Use Improved'

    - step: REPLACE_TEXT
      at: CURSOR
      in: '{{ flowContext.comparison }}'
```

### Pattern: Multi-Step Generation

```yaml
- title: 'Guided Article Creation'
  flow:
    # Step 1: Collect requirements
    - step: FORM
      form:
        topic:
          type: text
          label: 'Topic'
        tone:
          type: select
          label: 'Tone'
          options: [Professional, Casual]

    # Step 2: Generate outline
    - step: HUB_COMPLETION
      instruction: 'Create article outline about {{ flowContext.topic }}'
      response_format: 'list'

    # Step 3: Let user select sections
    - step: USER_SELECT
      promptText: 'Choose sections to include:'

    # Step 4: Generate full article
    - step: HUB_COMPLETION
      instruction: 'Write {{ flowContext.tone }} article about {{ flowContext.text }}'

    - step: INSERT_TEXT
      at: CURSOR
```

### Pattern: Batch Processing

```yaml
- title: 'Process Multiple Items'
  flow:
    # ...
    - step: FOR
      items: '{{ flowContext.paragraphList }}'
      var: currentParagraph
      do:
        - step: SHOW_PROGRESS
          message: 'Processing paragraph {{ flowContext.currentParagraph_index }}...'

        - step: HUB_COMPLETION
          instruction: 'Improve: {{ flowContext.currentParagraph }}'
```

## Anti-Patterns to Avoid

### Don't: Hardcode Environment-Specific Values

**Avoid:**

```yaml
SERVICES:
  HUB:
    endpoint: 'https://prod-server.company.com/swing/proactions'
    # Breaks in dev/test environments
```

**Do:**

```yaml
SERVICES:
  HUB:
    endpoint: '{BASE_URL}/swing/proactions'
    # Works in all environments
```

### Don't: Create God Actions

**Avoid:**

```yaml
- title: 'Do Everything'
  flow:
    # 50 steps doing unrelated things
```

**Do:**

```yaml
- title: 'Translate Text'
  # Focused on translation

- title: 'Generate Summary'
  # Focused on summarization
```
