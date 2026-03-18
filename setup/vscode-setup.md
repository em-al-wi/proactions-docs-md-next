# VSIDE and File Conventions

Configure Visual Studio Code for optimal ProActions development with schema validation, auto-completion, and proper file organization.

## File Naming Conventions

ProActions uses specific file naming conventions to enable automatic schema validation and better organization. These conventions are registered at [schemastore.org](https://schemastore.org) for automatic IDE support.

### Supported File Types

| File Type               | Description                 | Used For                               |
| ----------------------- | --------------------------- | -------------------------------------- |
| `.ai-kit.yaml`          | Main configuration file     | Primary ProActions configuration       |
| `.ai-kit.services.yaml` | Services configuration      | Service definitions (HUB, DEEPL, etc.) |
| `.ai-kit.menu.yaml`     | Menu configuration          | SLASH_MENU, ONECLICK_MENU              |
| `.ai-kit.section.yaml`  | Section configuration       | Individual sections in SECTIONS array  |
| `.ai-kit.template.yaml` | Template configuration      | Reusable step templates                |
| `.ai-kit.action.yaml`   | Single action configuration | Individual action steps                |
| `.ai-kit.step.yaml`     | Step configuration          | Individual workflow steps              |

### Example File Structure

```
/SysConfig/
├── pro-actions.ai-kit.yaml               # Main configuration
└── ProActions/
    ├── services.ai-kit.services.yaml     # Service definitions
    ├── oneclick.ai-kit.menu.yaml         # Oneclick menu
    ├── slash.ai-kit.menu.yaml            # Slash command menu
    └── sections/
        ├── headlines.ai-kit.section.yaml
        ├── translation.ai-kit.section.yaml
        └── content.ai-kit.section.yaml
```

## VS Code Configuration

### Install Required Extension

First, install the YAML extension by Red Hat:

1. Open VS Code
2. Go to Extensions (Ctrl+Shift+X / Cmd+Shift+X)
3. Search for "YAML" by Red Hat
4. Click Install

### Configure YAML Settings

Create or update `.vscode/settings.json` in your workspace:

```json
{
  "yaml.customTags": ["!include scalar"],
  "yaml.schemas": {
    "https://raw.githubusercontent.com/em-al-wi/proactions-schema/main/schema/ai-kit.schema.json": [
      "*.ai-kit.yaml",
      "*.ai-kit.yaml*"
    ],
    "https://raw.githubusercontent.com/em-al-wi/proactions-schema/main/schema/partial-services.schema.json": [
      "*.ai-kit.serv.yaml",
      "*.ai-kit.serv.yaml*",
      "*.ai-kit.services.yaml",
      "*.ai-kit.services.yaml*"
    ],
    "https://raw.githubusercontent.com/em-al-wi/proactions-schema/main/schema/partial-floatingMenu.schema.json": [
      "*.ai-kit.menu.yaml",
      "*.ai-kit.menu.yaml*"
    ],
    "https://raw.githubusercontent.com/em-al-wi/proactions-schema/main/schema/partial-templates.schema.json": [
      "*.ai-kit.templ.yaml",
      "*.ai-kit.templ.yaml*",
      "*.ai-kit.template.yaml",
      "*.ai-kit.template.yaml*"
    ],
    "https://raw.githubusercontent.com/em-al-wi/proactions-schema/main/schema/partial-section.schema.json": [
      "*.ai-kit.sect.yaml",
      "*.ai-kit.sect.yaml*",
      "*.ai-kit.section.yaml",
      "*.ai-kit.section.yaml*"
    ],
    "https://raw.githubusercontent.com/em-al-wi/proactions-schema/main/schema/partial-action.schema.json": [
      "*.ai-kit.action.yaml",
      "*.ai-kit.action.yaml*"
    ],
    "https://raw.githubusercontent.com/em-al-wi/proactions-schema/main/schema/partial-step.schema.json": [
      "*.ai-kit.step.yaml",
      "*.ai-kit.step.yaml*"
    ]
  }
}
```

### Configuration Explanation

**`yaml.customTags`**

- Enables support for the `!include` directive used in ProActions YAML files
- Required for modular configuration files

**`yaml.schemas`**

- Maps schema URLs to file patterns
- Provides validation and auto-completion for each file type
- Supports wildcard patterns (e.g., `*.ai-kit.yaml*` matches `test.ai-kit.yaml~backup`)

## Benefits of This Setup

### Schema Validation

- Real-time error detection in YAML files
- Validates step names, parameters, and configuration structure
- Prevents common configuration mistakes

### Auto-Completion

- IntelliSense for step names
- Parameter suggestions with descriptions
- Enum value completion for known options

### Documentation on Hover

- Hover over steps to see their documentation
- View parameter descriptions
- Quick reference without leaving the editor

## Splitting Configuration Files

The file naming conventions enable clean configuration splitting using `!include`:

### Main Configuration

```yaml
# pro-actions.ai-kit.yaml
AI_KIT:
  # Service Definitions
  SERVICES: !include /SysConfig/ProActions/services.ai-kit.services.yaml

  # Menu Configurations
  SLASH_MENU: !include /SysConfig/ProActions/slash.ai-kit.menu.yaml

  # Sections
  SECTIONS:
    - !include /SysConfig/ProActions/sections/headlines.ai-kit.section.yaml
    - !include /SysConfig/ProActions/sections/translation.ai-kit.section.yaml
```

### Service Configuration

```yaml
# services.ai-kit.services.yaml
HUB:
  endpoint: "{BASE_URL}/swing/proactions"
  target: openai
  defaultBehavior: You are a helpful assistant.

DEEPL:
  endpoint: /swing/proactions/proxy/forward/deepl
```

### Menu Configuration

```yaml
# text.ai-kit.menu.yaml
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
```

### Section Configuration

```yaml
# sections/headlines.ai-kit.section.yaml
section: Headlines
keywords: 'headline title'

actions:
  - title: 'Suggest Headlines'
    flow:
      - step: HUB_COMPLETION
        behavior: "Generate creative headlines"
        instruction: "Based on: {textContent}"
        response_format: 'list'

      - step: USER_SELECT
        promptText: 'Choose a headline:'

      - step: REPLACE_TEXT
        at: XPATH
        xpath: "/doc/story/grouphead/headline/p"
```

## Best Practices

### File Organization

1. **Keep main config minimal** - Use it only for top-level structure and includes
2. **Group by functionality** - One section per file
3. **Use descriptive names** - `headlines.ai-kit.section.yaml` not `sect1.yaml`
4. **Organize in folders** - Group related sections in subdirectories

### Naming Conventions

1. **Be consistent** - Always use the correct file suffix
2. **Use lowercase** - File names should be lowercase with hyphens
3. **Be descriptive** - Names should clearly indicate the file's purpose
4. **Avoid spaces** - Use hyphens instead of spaces in file names

## Troubleshooting

### Schema Not Working

**Problem:** No auto-completion or validation

**Solutions:**

1. Check YAML extension is installed and enabled
2. Verify `.vscode/settings.json` is in the workspace root
3. Reload VS Code window (Cmd/Ctrl+Shift+P → "Reload Window")
4. Check file naming matches the patterns exactly

### !include Not Recognized

**Problem:** VS Code shows errors for `!include` directive

**Solution:**
Add `"yaml.customTags": ["!include scalar"]` to settings

### Schema Outdated

**Problem:** Schema doesn't reflect latest steps

**Solution:**
The schema is automatically updated when ProActions is released. If you need the absolute latest:

1. Check [proactions-schema repository](https://github.com/em-al-wi/proactions-schema)
2. Clear VS Code's schema cache (Cmd/Ctrl+Shift+P → "Clear Editor History")
3. Reload the window

#

## Related Documentation

- [Configuration Basics](../guides/configuration/basics.md)
- [Installation](./installation.md)
