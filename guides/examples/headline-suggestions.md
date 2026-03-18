# Headline Suggestions

Generate AI-powered headline suggestions for articles using ProActions Hub.

## Overview

This example demonstrates how to:

- Send article content to an AI service for headline generation
- Use structured JSON output to get a list response (`response_format: 'list'`)
- Present suggestions to users with the USER_SELECT step
- Replace content at a specific XPath location

## Requirements

- ProActions Hub configured with an AI provider

## Use Case

Journalists often need multiple headline options to choose from. This workflow:

1. Reads the article content from the current document
2. Sends it to an AI service with instructions to generate headlines
3. Displays the suggestions to the user
4. Replaces the headline with the selected option

## Configuration

### Action Definition

Create a section file for headline-related actions:

```yaml
# /SysConfig/ProActions/sections/headlines.yaml
section: Headlines

actions:
  - title: 'Suggestions: Suggest alternative headlines'
    flow:
      # Show selection UI immediately while generating in background
      - step: USER_SELECT
        inlineSteps:
          # Generate headline suggestions using AI
          - step: HUB_COMPLETION
            behavior: |
              # Context
              You are an AI assistant helping journalists improve their content.

              # Task
              Your task is to suggest a maximum of 6 creative, engaging, and relevant headlines
              based on the article provided. The headlines should capture the essence of the
              article and be suitable for a news audience.

              Make sure to offer a variety of styles, such as informative, catchy, or
              thought-provoking options. The generated titles must not exceed 100 characters
              in length.

              Do not end the headlines with full stops and do not use double quotation marks
              unless you are quoting someone directly. Do not put the headlines in quotation
              marks! Do not hallucinate. Do not make up factual information.
            instruction: 'The article content is as follows: {textContent}'
            response_format: 'list'
        promptText: 'Suggested headlines:'
        enableKeyboardControl: true

      # Replace headline with selected option
      - step: REPLACE_TEXT
        at: XPATH
        xpath: "/doc/story/grouphead/headline/p"
```

**Note:** Using `inlineSteps` displays the selection modal immediately with a loading indicator while the AI generates headlines in the background. This eliminates perceived wait time and provides better user experience.

### OneClick Menu (Optional)

Add a contextual button that appears when editing headlines:

```yaml
# /SysConfig/ProActions/menu.oneclick.yaml
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

Note: To use `flowRef`, add an `id` to your action:

```yaml
actions:
  - id: suggest_headlines  # Add this ID for flowRef
    title: 'Suggestions: Suggest alternative headlines'
    flow:
      # ... rest of the configuration
```

### Main Configuration

Include the headline section in your main configuration:

```yaml
# pro-actions.ai-kit.yaml
AI_KIT:
  SERVICES: !include /SysConfig/ProActions/services.yaml
  ONECLICK_MENU: !include /SysConfig/ProActions/menu.oneclick.yaml
  SECTIONS:
    - !include /SysConfig/ProActions/sections/headlines.yaml
```

## How It Works

### Step 1: Generate Headlines

The HUB_COMPLETION step sends the article content to the AI service:

```yaml
- step: HUB_COMPLETION
  behavior: |
    You are an AI assistant helping journalists improve their content.
    Your task is to suggest 6 creative headlines...
  instruction: 'The article content is as follows: {textContent}'
  response_format: 'list'
```

**Key points:**

- `{textContent}` - Built-in variable containing the full document text
- `response_format: 'list'` - Returns an array of strings instead of plain text
- `behavior` - Provides context and constraints for the AI model

### Step 2: User Selection

The USER_SELECT step presents the list to the user:

```yaml
- step: USER_SELECT
  promptText: 'Suggested headlines:'
```

This automatically:

- Displays all items from the previous step's list output
- Allows the user to select one option
- Stores the selected text in `{selectedText}`

### Step 3: Replace Headline

The REPLACE_TEXT step updates the document:

```yaml
- step: REPLACE_TEXT
  at: XPATH
  xpath: "/doc/story/grouphead/headline/p"
```

This replaces the content at the specified XPath with `{selectedText}` from the previous step.

## Customization

### Adjust Number of Suggestions

Modify the behavior prompt to change the number of headlines:

```yaml
behavior: |
  Your task is to suggest a maximum of 10 creative headlines...
```

### Add Style Preferences

Customize the headline style:

```yaml
behavior: |
  Generate headlines in these styles:
  - 2 informative headlines (straightforward, factual)
  - 2 catchy headlines (engaging, attention-grabbing)
  - 2 thought-provoking headlines (questioning, analytical)
```

### Target Different Elements

Change the XPath to target different document elements:

```yaml
# For sub-headlines
xpath: "/doc/story/grouphead/subheadline/p"

# For different document types
xpath: "/doc/story/title"
```

## Best Practices

1. **Clear Instructions**: Provide specific constraints (character limits, style, tone)
2. **Content Quality**: The AI can only work with the content provided - ensure articles are complete
3. **User Choice**: Always let users select from suggestions rather than auto-replacing
4. **Context Awareness**: Use the OneClick menu to make the action available only in relevant contexts

## Troubleshooting

**No suggestions appear:**

- Check that ProActions Hub is configured correctly
- Verify the article has content (`{textContent}` is not empty)
- Check browser console for errors

**Suggestions are off-topic:**

- Ensure the article content is complete and relevant
- Refine the `behavior` prompt with more specific instructions
- Consider sending only the first few paragraphs instead of full content

**XPath doesn't work:**

- Verify the XPath matches your document structure
- Use browser dev tools to inspect the document XML structure
- Test the XPath in the browser console: `client.getElementByXPath("/doc/story/grouphead/headline/p")`

## Related Examples

- [Text Rewriting](./deepl-write.md) - Rewrite selected text with AI
- [Content Import and Rewrite](./content-import.md) - Import and transform external content
