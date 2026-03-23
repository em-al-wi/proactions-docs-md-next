# Configuration

This guide explains the initial steps to configure ProActions after installation. For comprehensive configuration documentation, see the [Configuration Guides](../guides/configuration/basics.md).

## Quick Start Configuration

After installing ProActions, you need to create configuration files to define services and actions.

## Minimum Required Configuration

At minimum, you need two configuration files:

1. **Main configuration file**: `/SysConfig/pro-actions.ai-kit.yaml`
2. **Service configuration file**: `/SysConfig/ProActions/services.ai-kit.services.yaml`

## Step 1: Create Main Configuration File

**File:** `/SysConfig/pro-actions.ai-kit.yaml`

```yaml
AI_KIT:
  # Include service configurations
  SERVICES:
    !include /SysConfig/ProActions/services.ai-kit.services.yaml

  # Optional: global reusable blocks (recommended for new configs)
  BLOCKS:
    constants:
      modulesBasePath: /SysConfig/ProActions/lib

  # Your action sections will go here
  SECTIONS:
    # We'll add sections in the next step
```

## Step 2: Configure Services

**File:** `/SysConfig/ProActions/services.ai-kit.services.yaml`

### Using ProActions Hub (Recommended)

```yaml
HUB:
  endpoint: "{BASE_URL}/swing/proactions"
  target: openai
  defaultBehavior: |
    You are an assistant who helps authors write their content.
    Answer without unnecessary explanations.
    Answer in English unless instructed otherwise.
```

**Why ProActions Hub?**

- Centralized API key management
- Secure - keys never exposed in configuration
- Support for 40+ AI providers
- See [Service Configuration Guide](../guides/configuration/service-configuration.md) for details

### Adding DeepL Translation

```yaml
HUB:
  endpoint: "{BASE_URL}/swing/proactions"
  target: openai
  defaultBehavior: |
    You are an assistant who helps authors write their content.

DEEPL:
  endpoint: /swing/proactions/proxy/forward/deepl
```

## Step 3: Create Your First Action Section

Create a new file for your actions:

**File:** `/SysConfig/ProActions/sections/my-first-actions.ai-kit.section.yaml`

```yaml
section: My Actions
keywords: 'getting started, test'

actions:
  - title: 'Test Action'
    flow:
      - step: SHOW_NOTIFICATION
        message: "ProActions is working!"
        notificationType: success
```

## Step 4: Include Your Section

Update your main configuration to include the new section:

**File:** `/SysConfig/pro-actions.ai-kit.yaml`

```yaml
AI_KIT:
  SERVICES:
    !include /SysConfig/ProActions/services.ai-kit.services.yaml

  SECTIONS:
    - !include /SysConfig/ProActions/sections/my-first-actions.ai-kit.section.yaml
```

## Step 5: Test Your Configuration

### In Development Mode

1. Refresh your browser (F5)
2. Press `Ctrl+K` (or `Cmd+K`)
3. Search for "Test Action"
4. Execute it to see the notification

### In Production

1. Clear Swing configuration cache
2. Refresh your browser
3. Test as above

## Recommended Directory Structure

Organize your configuration files for maintainability:

```
/SysConfig/
├── pro-actions.ai-kit.yaml           # Main entry point
└── ProActions/
    ├── services.ai-kit.services.yaml     # Service definitions
    └── sections/
        ├── content-tools.ai-kit.section.yaml        # Content-related actions
        ├── translation.ai-kit.section.yaml          # Translation actions
        └── my-custom-actions.ai-kit.section.yaml    # Your custom actions
```

## Configuration File Types

ProActions uses specific file naming conventions for automatic schema validation:

| File Pattern                                     | Purpose             |
| ------------------------------------------------ | ------------------- |
| `*.ai-kit.yaml`                                  | Main configuration  |
| `*.ai-kit.serv.yaml` or `*.ai-kit.services.yaml` | Service definitions |
| `*.ai-kit.sect.yaml` or `*.ai-kit.section.yaml`  | Section definitions |
| `*.ai-kit.action.yaml`                           | Action definitions  |
| `*.ai-kit.menu.yaml`                             | Menu configurations |

For complete file naming conventions and VS Code setup, see [VS Code Setup](./vscode-setup.md).

## Essential Configuration Concepts

### The !include Directive

Split your configuration into manageable files:

```yaml
AI_KIT:
  SERVICES:
    !include /SysConfig/ProActions/services.ai-kit.services.yaml

  SECTIONS:
    - !include /SysConfig/ProActions/sections/headlines.ai-kit.section.yaml
    - !include /SysConfig/ProActions/sections/translation.ai-kit.section.yaml
```

Path rules:

- Absolute paths are supported.
- Relative paths are supported only with `./` and `../`.
- Relative paths resolve against the file containing the `!include`.
- Bare non-dot paths keep existing behavior for compatibility.

```yaml
AI_KIT:
  SERVICES:
    !include ./ProActions/services.ai-kit.services.yaml

  SECTIONS:
    - !include ./ProActions/sections/headlines.ai-kit.section.yaml
```

**Benefits:**

- Better organization
- Easier maintenance
- Team collaboration
- Reusability across environments

### Reusable Blocks and Imports (Recommended)

For new configurations, define reusable blocks and module imports explicitly, then reference them by scope.

```yaml
AI_KIT:
  BLOCKS:
    constants:
      modulesBasePath: /SysConfig/ProActions/lib

  SECTIONS:
    - section: Editorial
      imports:
        - from: '{{ global.constants.modulesBasePath }}/common.ai-kit.module.yaml'
          as: common
      blocks:
        vars:
          normalizeFlowRef: global.flows.normalizeStory
      actions:
        - title: Normalize Story
          imports:
            - from: '{{ global.constants.modulesBasePath }}/editorial.ai-kit.module.yaml'
              as: editorial
          flow:
            - step: CALL_FLOW
              ref: '{{ section.vars.normalizeFlowRef }}'
```

Behavior notes:

- Section imports are available to all actions in that section.
- Action imports are merged with section imports.
- Alias conflicts between section and action imports resolve in favor of the action.
- Duplicate aliases in the same import list are rejected.
- Import cycles are rejected.

Import path rules:

- `imports[].from` supports absolute and dot-relative paths (`./`, `../`).
- Relative import paths resolve against the file that defines the import.

Migration tip:

- Existing `TEMPLATES` with `CALL_TEMPLATE.name` continue to work.
- Migrate gradually to `BLOCKS` + `CALL_FLOW.ref` and `SCRIPTING.scriptRef`.

### Built-in Variables

ProActions provides variables you can use immediately:

- `{textContent}` - Full document text
- `{selectedText}` - Selected text
- `{xmlContent}` - Full document XML
- `{userName}` - Current user's name
- `{BASE_URL}` - Your Swing instance URL

**Example:**

```yaml
- step: HUB_COMPLETION
  instruction: "Summarize: {textContent}"
```

For all variables, see [Working with Variables](../getting-started/working-with-variables.md).

## Quick Configuration Examples

### Example 1: Simple Text Improvement

```yaml
section: Quick Tools
actions:
  - title: 'Improve Selected Text'
    flow:
      - step: HUB_COMPLETION
        instruction: "Improve this text: {selectedText}"

      - step: REPLACE_TEXT
        at: CURSOR
```

### Example 2: Translation to French

```yaml
section: Translation
actions:
  - title: 'Translate to French'
    flow:
      - step: DEEPL_TRANSLATE
        instruction: "{selectedText}"
        target_lang: "FR"

      - step: REPLACE_TEXT
        at: CURSOR
```

### Example 3: Generate Headlines

```yaml
section: Content Generation
actions:
  - title: 'Generate Headline Ideas'
    flow:
      - step: HUB_COMPLETION
        instruction: "Generate 5 headline options for: {textContent}"
        response_format: "list"

      - step: USER_SELECT
        promptText: "Choose a headline:"

      - step: REPLACE_TEXT
        at: XPATH
        xpath: "/doc/story/grouphead/headline/p"
```

## Applying Configuration Changes

### Swing Development Mode (Recommended for Testing)

1. Save your YAML changes
2. Refresh browser (F5) or reload the editor
3. Changes apply immediately

### Production Mode

1. Save your YAML changes
2. Clear Swing configuration cache
3. Refresh browser
4. Changes take effect

## Common Configuration Issues

### Actions Don't Appear

**Check:**

- YAML syntax is valid (no tabs, proper indentation)
- Section is included in main configuration
- File paths are correct (absolute or dot-relative `./`/`../`)
- Browser cache is cleared

**Solution:**

```yaml
# Verify your section is included
AI_KIT:
  SECTIONS:
    - !include /SysConfig/ProActions/sections/my-actions.ai-kit.section.yaml
```

### Service Errors

**Check:**

- ProActions Hub is running (if using Hub)
- Service endpoint URLs are correct
- API keys are configured in Hub
- Network connectivity

**Test with:**

```yaml
- title: 'Test Service Connection'
  flow:
    - step: HUB_COMPLETION
      instruction: "Say hello"
      options:
        max_tokens: 10
```

### YAML Syntax Errors

**Check:**

- Use spaces, not tabs
- Proper indentation (2 or 4 spaces consistently)
- Quotes closed properly
- Colons followed by space

**Validate:**

- Check browser console (F12) for error messages
- Configure VS Code with ProActions schema (see [VS Code Setup](./vscode-setup.md))

## Security Best Practices

### Never Hardcode API Keys

**❌ Don't do this:**

```yaml
SERVICES:
  OPENAI:
    apiKey: "sk-proj-abc123..."  # Exposed!
```

**✅ Use ProActions Hub:**

```yaml
SERVICES:
  HUB:
    endpoint: "{BASE_URL}/swing/proactions"
    target: openai  # Keys managed securely in Hub
```

### Use Appropriate Permissions

Restrict sensitive actions to authorized users:

```yaml
section: Admin Tools
forGroups:
  - "Administrators"
  - "Editors"
actions:
  - title: 'Sensitive Action'
    flow:
      # ... workflow steps
```

## Next Steps

Now that you have basic configuration set up:

1. **[Create Your First Flow](../getting-started/your-first-flow.md)** - Build a complete workflow
2. **[Understanding Steps](../getting-started/understanding-steps.md)** - Learn about available steps
3. **[Configuration Basics Guide](../guides/configuration/basics.md)** - Deep dive into configuration
4. **[Service Configuration Guide](../guides/configuration/service-configuration.md)** - Advanced service setup

## Advanced Configuration Topics

For more advanced configuration, see these guides:

- **[Configuration Basics](../guides/configuration/basics.md)** - Sections, actions, permissions, context filtering
- **[Service Configuration](../guides/configuration/service-configuration.md)** - All service types and options
- **[Forms and Inputs](../guides/configuration/forms-and-inputs.md)** - Interactive form creation
- **[Menus and Triggers](../guides/configuration/menus-and-triggers.md)** - Text menus, one-click menus, instant actions

## Getting Help

If you encounter issues:

1. Check browser console (F12) for errors
2. Validate YAML syntax
3. Review [Troubleshooting Guide](../guides/howto/debug-flows.md)
4. Check [Configuration Basics](../guides/configuration/basics.md) for detailed explanations
