# Configuration Basics

This guide covers the fundamental concepts of ProActions configuration, helping you understand how to organize and structure your workflows effectively.

## Configuration Philosophy

ProActions follows a declarative configuration approach:

- Define **what** you want to happen, not **how**
- Use YAML for human-readable configuration
- Compose complex workflows from simple steps
- Reuse configurations across different actions

## Core Configuration Concepts

### 1. Hierarchical Structure

ProActions configuration follows a clear hierarchy:

```
AI_KIT
└── SERVICES (service definitions)
└── BLOCKS (global reusable constants/vars/flows/scripts)
└── SECTIONS (groups of actions)
    └── blocks (section-scoped reusable constants/vars/flows/scripts)
    └── imports (section-scoped module aliases)
    └── actions (individual workflows)
        └── blocks (action-local reusable constants/vars/flows/scripts)
        └── imports (action-local module aliases)
        └── flow (sequence of steps)
            └── step (individual operations)
```

### 2. File Organization

Organize your configuration files logically:

```
/SysConfig/
├── pro-actions.ai-kit.yaml             # Main entry point
└── ProActions/
    ├── services.ai-kit.services.yaml   # Service configurations
    ├── oneclick.ai-kit.menu.yaml       # One-click menu
    ├── text.ai-kit.menu.yaml           # Text selection menu
    └── sections/
        ├── content/
        │   ├── headlines.ai-kit.section.yaml
        │   └── summaries.ai-kit.section.yaml
        ├── translation/
        │   ├── deepl.ai-kit.section.yaml
        │   └── multilingual.ai-kit.section.yaml
        └── media/
            ├── images.ai-kit.section.yaml
            └── audio.ai-kit.section.yaml
```

### 3. Using !include

The `!include` directive allows modular configuration:

```yaml
AI_KIT:
  SERVICES:
    !include /SysConfig/ProActions/services.ai-kit.services.yaml

  TEXT_MENU:
    !include /SysConfig/ProActions/text.ai-kit.menu.yaml

  ONECLICK_MENU:
    !include /SysConfig/ProActions/oneclick.ai-kit.menu.yaml

  SECTIONS:
    # Content workflows
    - !include /SysConfig/ProActions/sections/content/headlines.ai-kit.section.yaml
    - !include /SysConfig/ProActions/sections/content/summaries.ai-kit.section.yaml

    # Translation workflows
    - !include /SysConfig/ProActions/sections/translation/deepl.ai-kit.section.yaml

    # Media workflows
    - !include /SysConfig/ProActions/sections/media/images.ai-kit.section.yaml
```

Path rules for `!include`:

- Absolute paths remain supported (for example `/SysConfig/ProActions/...`).
- Relative paths are supported only with `./` and `../`.
- Relative paths are resolved against the file that contains the `!include`.
- Bare non-dot paths (for example `SysConfig/...`) keep existing behavior for backward compatibility.

```yaml
AI_KIT:
  SERVICES:
    !include ./ProActions/services.ai-kit.services.yaml

  SECTIONS:
    - !include ./ProActions/sections/content/headlines.ai-kit.section.yaml
```

### 4. Reusable Blocks and Scoped Imports

For new configurations, prefer reusable blocks and explicit scope-based references.

```yaml
AI_KIT:
  BLOCKS:
    constants:
      modulesBasePath: /SysConfig/ProActions/lib
    flows:
      normalizeStory:
        - step: HUB_COMPLETION
          instruction: 'Normalize this article: {textContent}'

  SECTIONS:
    - section: Editorial
      imports:
        - from: '{{ global.constants.modulesBasePath }}/common.ai-kit.module.yaml'
          as: common
      blocks:
        vars:
          defaultFlowRef: global.flows.normalizeStory
      actions:
        - title: Normalize Story
          imports:
            - from: '{{ global.constants.modulesBasePath }}/editorial.ai-kit.module.yaml'
              as: editorial
          blocks:
            scripts:
              sumFromParams: params.a + params.b
          flow:
            - step: CALL_FLOW
              ref: '{{ section.vars.defaultFlowRef }}'
            - step: SCRIPTING
              scriptRef: local.scripts.sumFromParams
              returnAs: sum
```

Scope namespaces available in template resolution:

- `global` - `AI_KIT.BLOCKS`
- `section` - `section.blocks`
- `local` - `action.blocks`
- `imports.<alias>` - resolved import module blocks

Import behavior:

- Section imports are automatically available to every action in that section.
- Action imports are merged with section imports.
- If section and action declare the same alias, the action alias overrides for that action scope.
- Duplicate aliases in the same `imports` array are rejected.
- Circular module imports are rejected with an explicit cycle error.

Import path rules:

- `imports[].from` supports absolute paths.
- `imports[].from` also supports relative `./` and `../`.
- Relative import paths are resolved against the file that declares the import.
- Bare non-dot paths keep existing behavior for backward compatibility.

## Section Configuration

Sections group related actions together in the command palette.

### Basic Section

```yaml
section: Content Tools
actions:
  - title: 'Improve Headline'
    flow:
      - step: HUB_COMPLETION
        instruction: "Improve: {selectedText}"
      - step: REPLACE_TEXT
        at: CURSOR
```

### Section with Permissions

```yaml
section: Editorial Tools
forGroups:
  - "Editors"
  - "Senior Writers"
forTeams:
  - "NewsTeam"
actions:
  - title: 'Generate Headline Ideas'
    flow:
      # steps...
```

### Section with Context Filtering

```yaml
section: Story Tools
context: 'story'  # Only available in story context
app: 'swing'      # Only in Swing (not Prime)
actions:
  - title: 'Process Article'
    flow:
      # steps...
```

### Section Attributes Reference

| Attribute   | Purpose                                             | Example                        |
| ----------- | --------------------------------------------------- | ------------------------------ |
| `section`   | Section name                                        | `"Content Tools"`              |
| `keywords`  | Search keywords to find all actions of this section | `"writing content text"`       |
| `icon`      | Section icon as html (or svg)                       | `"<i class='fa fa-edit'></i>"` |
| `disabled`  | Disable section                                     | `true`                         |
| `hidden`    | Hide from palette, still executable from API        | `true`                         |
| `context`   | Limit to context                                    | `report'`, `'dashboard'`       |
| `app`       | Limit to application                                | `'swing'`, `'prime'`           |
| `forUsers`  | User permissions                                    | `["user1", "user2"]`           |
| `forGroups` | Group permissions                                   | `["Editors"]`                  |
| `forTeams`  | Team permissions                                    | `["NewsTeam"]`                 |

## Action Configuration

Actions are individual workflows users can execute.

### Basic Action

```yaml
- title: 'Translate to French'
  flow:
    - step: DEEPL_TRANSLATE
      instruction: "{selectedText}"
      target_lang: "FR"

    - step: REPLACE_TEXT
      at: CURSOR
```

### Action with Conditional Availability

Only show action when a specific element is selected:

```yaml
- title: 'Anonymize Image'
  isAvailable: "{{ client.getSelectedElement() === 'web-image' }}"
  flow:
    - step: SELECTED_OBJECT

    - step: BRIGHTER_AI
```

### Action with User Input

Collect input before executing workflow:

```yaml
- title: 'Generate Article from Topic'
  flow:
    - step: PROMPT
      promptText: "Enter article topic:"

    - step: HUB_COMPLETION
      instruction: "Write a news article about: {{ flowContext.text }}"

    - step: INSERT_TEXT
      at: CURSOR
```

### Instant Action

Execute action with keyboard shortcut based on cursor position:

```yaml
- title: 'Quick Improve'
  instantActionContext: "/doc/story/body/p,/doc/story/headline/p"
  flow:
    - step: HUB_COMPLETION
      instruction: "Improve: {selectedText}"

    - step: REPLACE_TEXT
      at: CURSOR
```

### Action Attributes Reference

| Attribute              | Purpose                                           | Example                                       |
| ---------------------- | ------------------------------------------------- | --------------------------------------------- |
| `title`                | Action name                                       | `"Translate to French"`                       |
| `keywords`             | Search keywords to find the action in the palette | `"translation, french"`                       |
| `icon`                 | Action icon                                       | `"<i class='fa fa-language'></i>"`            |
| `disabled`             | Disable action                                    | `true`                                        |
| `hidden`               | Hide from palette                                 | `true`                                        |
| `isAvailable`          | Availability condition, templates supported       | `" {{ client.getSelectedElement() === 'p' }"` |
| `instantActionContext` | Instant action XPaths                             | `"/doc/story/body/p"`                         |
| `context`              | Limit to context                                  | `'story'`                                     |
| `app`                  | Limit to application                              | `'swing'` or `'prime'`                        |
| `forUsers`             | User permissions                                  | `["user1"]`                                   |
| `forGroups`            | Group permissions                                 | `["Editors"]`                                 |
| `forTeams`             | Team permissions                                  | `["News"]`                                    |
| `panelContext`         | Prime panel context                               | (see Prime configuration)                     |
| `category`             | Prime sub-category                                | `"Content"`                                   |

## Flow Configuration

Flows define the sequence of steps that execute when an action runs.

### Linear Flow

Steps execute in order:

```yaml
flow:
  - step: GET_TEXT_CONTENT
    at: CURSOR

  - step: HUB_COMPLETION
    instruction: "Summarize: {selectedText}"

  - step: SHOW_RESPONSE
```

### Flow with Conditional Logic

```yaml
flow:
  - step: GET_TEXT_CONTENT
    at: CURSOR

  - step: IF
    condition: "{{ flowContext.text }}"
    then:
      - step: HUB_COMPLETION
        instruction: "Process: {{ flowContext.text }}"
      - step: REPLACE_TEXT
        at: CURSOR
    else:
      - step: SHOW_NOTIFICATION
        message: "Please select text first"
```

### Flow with Error Handling

```yaml
flow:
  - step: TRY
    try:
      - step: HUB_COMPLETION
        instruction: "Analyze: {textContent}"

      - step: SHOW_RESPONSE
    catch:
      - step: SHOW_NOTIFICATION
        notificationType: error
        message: "An error occurred. Please try again."
```

### Flow with User Choice

```yaml
flow:
  - step: HUB_COMPLETION
    instruction: "Generate 5 headline options for: {textContent}"
    response_format: "list"

  - step: USER_SELECT
    promptText: "Choose a headline:"

  - step: REPLACE_TEXT
    at: XPATH
    xpath: "/doc/story/headline/p"
```

## Permission System

ProActions supports three levels of permissions:

### User-Level Permissions

```yaml
section: Admin Tools
forUsers:
  - "admin"
  - "editor_chief"
actions:
  # Only visible to specified users
```

### Group-Level Permissions

```yaml
section: Editor Tools
forGroups:
  - "Editors"
  - "Senior Editors"
actions:
  # Only visible to group members
```

### Team-Level Permissions

```yaml
section: News Team Tools
forTeams:
  - "News Team"
  - "Breaking News Team"
actions:
  # Only visible to team members
```

### Combined Permissions

```yaml
section: Premium Tools
forGroups:
  - "Editors"
forTeams:
  - "Premium Content Team"
actions:
  # Visible to users in EITHER Editors group OR Premium Content Team
```

## Context Filtering

Limit actions to specific contexts:

### Story Context

```yaml
section: Story Tools
context: 'story'
actions:
  - title: 'Process Story'
    flow:
      # Only available when editing stories
```

### Dashboard Context

```yaml
section: Dashboard Actions
context: 'dashboard'
actions:
  - title: 'Bulk Operations'
    flow:
      # Only available in dashboard
```

### All Contexts

```yaml
section: Universal Tools
context: 'all'  # Default
actions:
  # Available everywhere
```

### Available Contexts

- `'all'` - All contexts (default)
- `'story'` - Story editor
- `'dwp'` - DWP (Digital Work Package)
- `'report'` - Report context
- `'dashboard'` - Dashboard view

## Application Filtering

Limit to specific applications:

### Swing Only

```yaml
section: Swing Features
app: 'swing'
actions:
  - title: 'Swing-Specific Action'
    flow:
      # Only in Swing Web Client
```

### Prime Only

```yaml
section: Prime Features
app: 'prime'
actions:
  - title: 'Prime-Specific Action'
    flow:
      # Only in Prime Windows Client
```

## Variables and Data Flow

### Built-in Variables

Available everywhere with single braces:

```yaml
- step: HUB_COMPLETION
  instruction: "Process: {textContent}"    # Full document text

- step: HUB_COMPLETION
  instruction: "Improve: {selectedText}"   # Selected text

- step: HUB_COMPLETION
  instruction: "User: {userName}"          # Current user
```

### Custom Variables

Set and use with double braces:

```yaml
- step: SET
  myVariable: "Custom value"
  anotherVar: "{userName}'s workspace"

- step: SHOW_NOTIFICATION
  message: "Value: {{ flowContext.myVariable }}"
```

### Template Expressions

Access client methods:

```yaml
- step: SET
  title: "{{ client.getMetadata().metadata.general.title }}"
  workFolder: "{{ client.getDocumentWorkfolder() }}"
```

## Configuration Best Practices

### 1. Use Descriptive Names

```yaml
# Good
- title: 'Generate SEO-Optimized Headlines'

# Avoid
- title: 'Gen Heads'
```

### 2. Group Related Actions

```yaml
section: Content Enhancement
actions:
  - title: 'Improve Headline'
  # ...
  - title: 'Enhance Introduction'
  # ...
  - title: 'Optimize Summary'
  # ...
```

### 3. Add Keywords for Discoverability

```yaml
- title: 'Translate to French'
  keywords: 'translation france langue traduire francais'
```

### 4. Use Meaningful Flow Variable Names

```yaml
# Good
- step: SET
  selectedArticleText: "{selectedText}"
  translationTargetLanguage: "FR"

# Avoid
- step: SET
  txt: "{selectedText}"
  lang: "FR"
```

### 5. Handle Errors Gracefully

```yaml
- step: TRY
  try:
    - step: HUB_COMPLETION
      instruction: "Process: {textContent}"
  catch:
    - step: SHOW_NOTIFICATION
      message: "Unable to process. Please try again."
```

## Common Patterns

### Pattern 1: Text Enhancement

```yaml
- title: 'Enhance Text'
  flow:
    - step: HUB_COMPLETION
      instruction: "Improve: {selectedText}"
    - step: REPLACE_TEXT
      at: CURSOR
```

### Pattern 2: Generate and Choose

```yaml
- title: 'Generate Options'
  flow:
    - step: HUB_COMPLETION
      instruction: "Generate 5 options for: {selectedText}"
      response_format: "list"
    - step: USER_SELECT
      promptText: "Choose one:"
    - step: REPLACE_TEXT
      at: CURSOR
```

### Pattern 3: Collect Input and Process

```yaml
- title: 'Custom Generation'
  flow:
    - step: PROMPT
      promptText: "Enter your requirements:"
    - step: HUB_COMPLETION
      instruction: "Generate based on: {{ flowContext.text }}"
    - step: INSERT_TEXT
      at: CURSOR
```

### Pattern 4: Conditional Processing

```yaml
- title: 'Smart Processing'
  flow:
    - step: IF
      condition: "{{ client.isTextSelected() }}"
      then:
        - step: HUB_COMPLETION
          instruction: "Process: {selectedText}"
        - step: REPLACE_TEXT
          at: CURSOR
      else:
        - step: PROMPT
          promptText: "Enter text to process:"
        - step: HUB_COMPLETION
          instruction: "Process: {{ flowContext.text }}"
        - step: INSERT_TEXT
          at: CURSOR
```

## Next Steps

- **[Service Configuration](./service-configuration.md)** - Configure external services
- **[Forms and Inputs](./forms-and-inputs.md)** - Create interactive forms
- **[Menus and Triggers](./menus-and-triggers.md)** - Configure menus and shortcuts
