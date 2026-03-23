# Your First Flow

This guide will walk you through creating your first ProActions workflow. We'll create a simple action that suggests alternative headlines for an article using AI.

## Prerequisites

Before you begin, ensure that:

- ProActions is installed and configured in your Swing or Prime environment
- ProActions Hub is set up with an OpenAI API target (for secure API key management)
- You have access to edit configuration files in `/SysConfig/ProActions/`

## Understanding the Flow Structure

A ProActions flow consists of:

1. **Section**: A group of related actions
2. **Action**: A specific task with a title and a flow
3. **Flow**: A sequence of steps that execute in order
4. **Steps**: Individual operations (like calling an API, showing a prompt, or inserting text)

## Creating Your First Action

Let's create an action that suggests headline alternatives for a news article.

### Step 1: Create the Configuration File

Create a new YAML file at `/SysConfig/ProActions/sections/my-first-actions.ai-kit.section.yaml`:

```yaml
section: My First Actions
actions:
  - title: 'Suggest Headlines'
    flow:
      # Step 1: Get the article content and send it to AI
      - step: HUB_COMPLETION
        behavior: |
          You are an AI assistant helping authors improve their content.
          Suggest 5 creative, engaging headlines based on the article provided.
          Headlines should be under 100 characters.
          Do not use quotation marks unless quoting someone directly.
        instruction: 'The article content is: {textContent}'
        response_format: 'list'

      # Step 2: Show the suggestions to the user
      - step: USER_SELECT
        promptText: 'Choose a headline:'

      # Step 3: Insert the selected headline
      - step: REPLACE_TEXT
        at: XPATH
        xpath: "/doc/story/grouphead/headline/p"
```

### Step 2: Include the Section in Main Configuration

Edit `/SysConfig/pro-actions.ai-kit.yaml` to include your new section:

```yaml
AI_KIT:
  SERVICES:
    HUB:
      endpoint: "{BASE_URL}/swing/proactions"
      target: openai

  SECTIONS:
    - !include /SysConfig/ProActions/sections/my-first-actions.ai-kit.section.yaml
```

### Step 3: Refresh the Configuration

**In development mode:**

- Open Swing/Prime
- Refresh the editor context (F5 or reload)

**In production:**

- Clear the Swing configuration cache
- Refresh the browser

### Step 4: Test Your Action

1. Open an article in the Swing or Prime editor
2. Press `Ctrl+K` (or `Cmd+K` on Mac) to open ProActions
3. Type "Suggest Headlines" to find your action
4. Press Enter to execute

The flow will:

1. Send your article content to the AI service via ProActions Hub
2. Display a list of suggested headlines
3. Replace the current headline with your selection

## Understanding Each Step

### Step 1: HUB_COMPLETION

```yaml
- step: HUB_COMPLETION
  behavior: |
    You are an AI assistant...
  instruction: 'The article content is: {textContent}'
  response_format: 'list'
```

- **step**: `HUB_COMPLETION` - Calls an LLM through ProActions Hub
- **behavior**: System prompt that defines the AI's role and instructions
- **instruction**: User prompt that includes the article content via `{textContent}` variable
- **response_format**: `'list'` tells the AI to return structured JSON as a list

The step automatically stores the result in the flow context, making it available for the next step.

### Step 2: USER_SELECT

```yaml
- step: USER_SELECT
  promptText: 'Choose a headline:'
```

- Shows a selection dialog with the list returned from the previous step
- The selected item is automatically stored and passed to the next step

### Step 3: REPLACE_TEXT

```yaml
- step: REPLACE_TEXT
  at: XPATH
  xpath: "/doc/story/grouphead/headline/p"
```

- Replaces text at the specified XPath location
- Uses the output from the previous step (the selected headline)

## Built-in Variables

ProActions provides several built-in variables you can use with single curly braces `{}`:

- `{textContent}` - Full text content of the document
- `{selectedText}` - Currently selected text in the editor
- `{xmlContent}` - Full XML content of the document
- `{userName}` - Current logged-in user's username

## Custom Variables / Templates

To access custom variables stored in the flow context, use the template syntax with double curly braces:

```yaml
- step: SET
  myVariable: "Hello World"

- step: SHOW_NOTIFICATION
  message: "The value is: {{ flowContext.myVariable }}"
```

## Next Steps

Now that you've created your first flow, you can:

- Learn about [Understanding Steps](./understanding-steps.md) to discover all available step types
- Explore [Working with Variables](./working-with-variables.md) to use dynamic data
- Read about [Using Services](./using-services.md) to integrate with external APIs

## Troubleshooting

**Action doesn't appear in ProActions menu:**

- Check that the YAML syntax is valid
- Ensure the section is properly included in `pro-actions.ai-kit.yaml`
- Clear browser cache and reload

**AI service not responding:**

- Verify ProActions Hub is configured correctly
- Check that API keys are set up in ProActions Hub
- Review browser console for error messages

**Text not replacing:**

- Verify the XPath matches your document structure
- Check that you're in the correct editor context
