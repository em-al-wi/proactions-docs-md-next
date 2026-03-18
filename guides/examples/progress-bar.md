# Progress Bar for Long Operations

Display progress indicators during multi-step workflows.

## Overview

This example demonstrates how to:

- Use SHOW_PROGRESS, UPDATE_PROGRESS, and HIDE_PROGRESS steps
- Perform numerical calculations in Nunjucks templates
- Use WHILE loops for iterative processing
- Provide user feedback during long-running operations

## Requirements

- No external services required
- ProActions AI-Kit 1.0.8 or later

## Use Case

Long-running workflows can leave users wondering if the system is still working. Progress bars provide visual feedback and improve user experience by:

- Showing that processing is ongoing
- Indicating how much work remains
- Displaying status messages
- Preventing users from interrupting operations prematurely

## Configuration

```yaml
# Example in pro-actions.ai-kit.yaml
section: Examples

actions:
  - title: 'Progress Example'
    flow:
      # Initialize variables
      - step: SET
        messagePrompt: "Create a short (about 5-10 words) reassuring message to the user that the current loading activity will be completed soon. Be creative!"
        completion: 0

      # Show the progress bar
      - step: SHOW_PROGRESS
        position: bottom

      # Process in steps until completion
      - step: WHILE
        condition: "{{ flowContext.completion < 100 }}"
        do:
          # Increase percentage
          - step: SET
            completion: "{{ flowContext.completion + 25 }}"
            convertTo: number

          # Use AI to generate a status message
          - step: HUB_COMPLETION
            instruction: "{{ flowContext.messagePrompt }}"

          # Wait a moment
          - step: SLEEP

          # Update the progress bar
          - step: UPDATE_PROGRESS
            percentage: "{{ flowContext.completion }}"

      # Show full progress bar for a moment
      - step: SLEEP
```

## How It Works

### Step 1: Initialize Variables

Set up the initial state:

```yaml
- step: SET
  messagePrompt: "Create a short reassuring message..."
  completion: 0
```

**Key points:**

- `messagePrompt` - Template for AI-generated messages
- `completion` - Tracks progress from 0 to 100

### Step 2: Show Progress Bar

Display the progress indicator:

```yaml
- step: SHOW_PROGRESS
  position: bottom
```

**Options for `position`:**

- `bottom` - Bottom of the screen (default)
- `top` - Top of the screen
- `center` - Center of the screen

### Step 3: Update Progress in Loop

Use WHILE loop to incrementally update:

```yaml
- step: WHILE
  condition: "{{ flowContext.completion < 100 }}"
  do:
    # Increase by 25% each iteration
    - step: SET
      completion: "{{ flowContext.completion + 25 }}"
      convertTo: number

    # Update the progress bar
    - step: UPDATE_PROGRESS
      percentage: "{{ flowContext.completion }}"
```

**Numerical calculation in templates:**

```yaml
"{{ flowContext.completion + 25 }}"  # Addition
"{{ flowContext.total - flowContext.processed }}"  # Subtraction
"{{ flowContext.count * 2 }}"  # Multiplication
"{{ flowContext.total / flowContext.parts }}"  # Division
```

**Type conversion:**

```yaml
- step: SET
  count: "{{ flowContext.stringNumber }}"
  convertTo: number  # Ensures it's stored as a number
```

### Step 4: Add Status Messages

Show what's happening:

```yaml
- step: UPDATE_PROGRESS
  percentage: "{{ flowContext.completion }}"
  status: "Processing item {{ flowContext.currentItem }}..."
```

### Step 5: Hide Progress

Remove the progress bar when complete:

```yaml
- step: HIDE_PROGRESS
```

Or it will auto-hide when the flow completes.

## Real-World Examples

### Multi-Step Workflow

Track progress through distinct phases:

```yaml
- title: 'Article Analysis Pipeline'
  flow:
    - step: SHOW_PROGRESS

    # Phase 1: Extract content (0-20%)
    - step: UPDATE_PROGRESS
      percentage: 10
      status: "Extracting article content..."

    - step: SET
      text: '{{ client.getTextContent() }}'

    - step: UPDATE_PROGRESS
      percentage: 20
      status: "Content extracted"

    # Phase 2: Analyze sentiment (20-40%)
    - step: UPDATE_PROGRESS
      percentage: 25
      status: "Analyzing sentiment..."

    - step: HUB_COMPLETION
      instruction: "Analyze sentiment in JSON: {{ flowContext.text }}"
      response_format: json_object
      outputs:
        - type: object
          name: sentiment

    - step: UPDATE_PROGRESS
      percentage: 40
      status: "Sentiment analyzed"

    # Phase 3: Extract topics (40-60%)
    - step: UPDATE_PROGRESS
      percentage: 45
      status: "Extracting topics..."

    - step: HUB_COMPLETION
      instruction: "Extract main topics: {{ flowContext.text }}"
      response_format: "list"
      outputs:
        - type: list
          name: topics

    - step: UPDATE_PROGRESS
      percentage: 60
      status: "Topics extracted"

    # Phase 4: Generate summary (60-80%)
    - step: UPDATE_PROGRESS
      percentage: 65
      status: "Generating summary..."

    - step: HUB_COMPLETION
      instruction: "Summarize: {{ flowContext.text }}"
      outputs:
        - type: text
          name: summary

    - step: UPDATE_PROGRESS
      percentage: 80
      status: "Summary generated"

    # Phase 5: Create report (80-100%)
    - step: UPDATE_PROGRESS
      percentage: 85
      status: "Creating report..."

    - step: SET
      text: |
        # Article Analysis Report

        **Sentiment:** {{ flowContext.sentiment.sentiment }}

        **Topics:** {{ flowContext.topics | join(', ') }}

        **Summary:** {{ flowContext.summary }}

    - step: UPDATE_PROGRESS
      percentage: 100
      status: "Report complete!"

    - step: SLEEP

    - step: MARKDOWN_TO_HTML

    - step: SHOW_RESPONSE
      mode: html

    - step: HIDE_PROGRESS
```

## Best Practices

1. **Meaningful Messages**: Show what's actually happening, not just percentages
2. **Accurate Percentages**: Base percentages on actual progress, not guesses
3. **Clean Up**: Always hide progress when done (or let flow complete)
4. **Sleep Before Hide**: Add small delay so users can see 100% completion
5. **Error States**: Update progress even when errors occur

## Troubleshooting

**Progress bar doesn't appear:**

- Check that SHOW_PROGRESS is called
- Verify no JavaScript errors in console
- Ensure ProActions UI components are loaded

**Progress doesn't update:**

- Verify UPDATE_PROGRESS is inside the loop
- Check that percentage variable is numeric
- Ensure percentage is between 0-100

**Progress bar freezes:**

- Check for infinite loops in WHILE conditions
- Add SLEEP steps to allow UI updates
- Verify loop termination conditions

**Percentage calculation wrong:**

- Use `convertTo: number` for numeric operations
- Check division by zero
- Round percentages: `{{ (value * 100) | round }}`

## Related Examples

- [Text-to-Speech](./text-to-speech.md) - Shows progress during audio generation
- [Image Outpainting](./image-outpainting.md) - Progress for image processing
- [Content Import](./content-import.md) - Multi-step import with progress
