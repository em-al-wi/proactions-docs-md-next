# Working with Variables

Variables are essential for passing data between steps in ProActions workflows. This guide explains how to use variables, the flow context, and dynamic data in your flows.

## The Flow Context

The **flow context** is a data store that persists throughout the execution of a flow. Each step can:

- Read variables from the flow context
- Write new variables to the flow context
- Modify existing variables

Think of it as a shared workspace where steps exchange data.

## Variable Syntax: Two Types

ProActions uses two different variable syntaxes depending on what you're accessing:

### Built-in Variables (Single Braces)

Built-in variables provided by ProActions use single curly braces `{}`:

```yaml
- step: HUB_COMPLETION
  instruction: "Summarize: {textContent}"
```

### Custom Variables (Double Braces with flowContext)

Custom variables stored in the flow context use double curly braces with `flowContext` prefix:

```yaml
- step: SET
  myVariable: "Hello World"

- step: SHOW_NOTIFICATION
  message: "Value: {{ flowContext.myVariable }}"
```

## Scoped Reusable Namespaces

When using reusable blocks (`BLOCKS`, `section.blocks`, `action.blocks`, and `imports`), template resolution also exposes scoped namespaces:

- `global` - reusable blocks defined in `AI_KIT.BLOCKS`
- `section` - reusable blocks defined in `section.blocks`
- `local` - reusable blocks defined in `action.blocks`
- `imports.<alias>` - reusable blocks from imported modules

### Scoped Constants and Vars

```yaml
AI_KIT:
  BLOCKS:
    constants:
      modulesBasePath: /SysConfig/ProActions/lib

  SECTIONS:
    - section: Editorial
      blocks:
        vars:
          endpoint: '{{ global.constants.modulesBasePath }}/api'
      actions:
        - title: Show Endpoint
          flow:
            - step: SHOW_NOTIFICATION
              message: '{{ section.vars.endpoint }}'
```

### Scoped References in Flow and Script Reuse

```yaml
- step: CALL_FLOW
  ref: '{{ section.vars.defaultFlowRef }}'

- step: SCRIPTING
  scriptRef: local.scripts.normalizeTitle
  returnAs: normalizedTitle
```

### Lazy Vars Behavior

Scoped `vars` are evaluated lazily at access time. This means they can reference current `flowContext` values, and changes in `flowContext` are reflected when the var is resolved later in the flow.

## Built-in Variables

ProActions automatically provides several built-in variables based on the current context:

### Document Variables

- `{textContent}` - Full text content of the current document
- `{xmlContent}` - Full XML content of the current document
- `{selectedText}` - Currently selected text in the editor
- `{selectedXml}` - XML of the current selection
- `{paragraphText}` - The text of the paragraph in which the cursor is located.
- `{paragraphXML}` - The XML of the paragraph in which the cursor is located.
- `{documentId}` - The ID of the currently opened document
- `{documentLoid}` - The LOID of the currently opened document
- `{documentUuid}` - The UUID of the currently opened document
- `{documentName}` - The name of the currently opened document
- `{issueDate}` - The issue date of the currently opened document
- `{reportId}` - The id of the currently opened report
- `{reportLoid}` - The LOID of the currently opened report
- `{reportUuid}` - The UUID of the currently opened report
- `{prompt}` - The current prompt text (for prompt-related operations)

### System Variables

- `{BASE_URL}` - The base URL of your Swing/Prime instance
- `{userName}` - Current logged-in user's username
- `{userId}` - Current user's ID
- `{appName}` - prime or swing
- `{edapiUrl}` - The baseUrl of Swing's EDAPI
- `{edapiToken}` - The user's EDAPI token

### Example Usage

```yaml
- step: HUB_COMPLETION
  behavior: "You are a helpful assistant."
  instruction: "Summarize this text: {textContent}"
```

## Setting Variables with SET

The `SET` step is the primary way to create and assign variables:

### Simple Assignment

```yaml
- step: SET
  myVariable: "Hello World"
  anotherVar: 42
  isActive: true
```

### Using Expressions

You can combine built-in and custom variables:

```yaml
- step: SET
  greeting: "Hello, {userName}!"
  fullMessage: "{{ flowContext.greeting }} Welcome back!"
```

### Using Template Syntax

Access client functions and perform complex operations:

```yaml
- step: SET
  text: '{{ client.getTextContent({ excludeTags: ["caption", "credit"] }) | safe }}'
  filename: "{{ client.getMetadata().metadata.general.title }}.mp3"
```

### Executing Steps to Set Variables

```yaml
- step: SET
  name: headline
  steps:
    - step: GET_TEXT_CONTENT
      at: XPATH
      xpath: "/doc/story/headline/p"
```

## Reading Variables

### From Built-in Sources

```yaml
- step: SHOW_NOTIFICATION
  message: "Selected text: {selectedText}"
```

### From Flow Context

```yaml
- step: SET
  userName: "John Doe"

- step: SHOW_NOTIFICATION
  message: "User: {{ flowContext.userName }}"
```

### Nested Properties

Access object properties using dot notation:

```yaml
- step: SET
  myObject:
    attrib1: "content"

- step: SHOW_NOTIFICATION
  message: "Object name: {{ flowContext.myObject.attrib1 }}"
  notificationType: success
```

### Array Access

Access array elements by index:

```yaml
- step: SET
  myArray: ["asd", "qwe"]

- step: SHOW_NOTIFICATION
  message: "Object name: {{ flowContext.myArray[0] }}"
  notificationType: success
```

## How Steps Pass Data

Steps automatically store their results in the flow context. The specific variable names depend on each step's implementation:

### Example: Automatic Data Flow

```yaml
# GET_TEXT_CONTENT stores result in flow context
- step: GET_TEXT_CONTENT
  at: CURSOR

# HUB_COMPLETION can read text (from previous step)
# and stores its result for the next step
- step: HUB_COMPLETION
  instruction: "Improve: {{ flowContext.text }}"

# REPLACE_TEXT uses the completion result automatically
- step: REPLACE_TEXT
  at: CURSOR
```

### Example: Named Outputs

Some steps support custom output names via `outputs`:

```yaml
- step: SELECTED_OBJECT
  outputs:
    - type: object
      name: myObject

- step: SHOW_NOTIFICATION
  message: "ID: {{ flowContext.myObject.id }}"
```

## Conditional Variables

Use variables in conditional statements:

### Simple Conditions

```yaml
- step: IF
  condition: "{{ client.isTextSelected() }}"
  then:
    - step: SHOW_NOTIFICATION
      message: "You have selected text"
```

### Comparison Operators

```yaml
- step: SET
  wordCount: 150

- step: IF
  condition: "{{ flowContext.wordCount > 100 }} "
  then:
    - step: SHOW_NOTIFICATION
      message: "Long article detected"
```

### Logical Operators

```yaml
- step: IF
  condition: "{{ client.isTextSelected() && flowContext.otherVar }}"
  then:
    - step: HUB_COMPLETION
      instruction: "Improve: {selectedText}"
```

## Working with Lists

### Creating Lists

```yaml
- step: SET
  myList:
    - "Item 1"
    - "Item 2"
    - "Item 3"
```

### Iterating Over Lists

```yaml
- step: FOR
  items: "{{ flowContext.myList }}"
  var: currentItem
  do:
    - step: SHOW_NOTIFICATION
      message: "Processing: {{ flowContext.currentItem }}"
```

### List from AI Response

```yaml
- step: HUB_COMPLETION
  instruction: "Generate 5 topic ideas about {selectedText}"
  response_format: "list"

- step: USER_SELECT
  promptText: "Choose a topic:"
```

## Working with Objects

### Creating Objects

```yaml
- step: SET
  userInfo:
    name: "{{ client.getUserName() }}"
    role: "editor"
```

### Accessing Object Properties

```yaml
- step: SHOW_NOTIFICATION
  message: "User {{ flowContext.userInfo.name }} is an {{ flowContext.userInfo.role }}"
```

### Parsing JSON

```yaml
- step: SET
  text: '{"title": "My Article", "author": "John Doe"}'

- step: PARSE_JSON

- step: SHOW_NOTIFICATION
  message: "Article by {{ flowContext.object.author }}"
```

## Advanced: Scripting with Variables

For complex logic, use the `SCRIPTING` step:

### JavaScript Expressions

```yaml
- step: SCRIPTING
  template: |
    flowContext.wordCount = flowContext.textContent.split(' ').length;
    flowContext.readingTime = Math.ceil(flowContext.wordCount / 200);
    return flowContext; // always return the flowContext !
```

### Script with Multiple Operations

```yaml
- step: SCRIPTING
  script: |
    const text = client.getSelectedText() || '';
    const words = text.split(' ');
    const uniqueWords = [...new Set(words)];

    flowContext.wordCount = words.length;
    flowContext.uniqueWordCount = uniqueWords.length;

    return flowContext;
```

## Template Filters and Functions

The template syntax supports filters and client functions:

### Common Filters

```yaml
- step: SET
  # String manipulation
  uppercased: "{{ flowContext.text | upper }}"
  truncated: "{{ flowContext.longText | truncate(50) }}"

  # Safe output (no escaping)
  content: "{{ client.getTextContent() | safe }}"

  # Regex replacement
  cleaned: "{{ flowContext.filename | replace(r/[\\/:?]/g, '_') }}"
```

### Custom Resolver Filters

Use these filters when you need to traverse arrays/objects inside `flowContext` data:

```yaml
- step: SET
  users:
    - id: 1
      profile:
        name: "Ada"
    - id: 2
      profile:
        name: "Linus"

  # Chain find + getAttr for targeted lookup
  selectedName: "{{ flowContext.users | find('id', 2) | getAttr('profile') | getAttr('name') | default('Unknown') }}"

  # Collect one nested path from all items
  namesCsv: "{{ flowContext.users | pluck('profile.name') | join(', ') }}"
```

Examples below use standard Nunjucks filter syntax.

| Filter | Description | Example Usage |
| --- | --- | --- |
| `getAttr(key)` | Returns one own property by key from an object. Returns empty output when missing. | `&#123;&#123; flowContext.user | getAttr('name') &#125;&#125;` |
| `find(key, value)` | Returns the first array item where own property `key` strictly equals `value`. | `&#123;&#123; flowContext.users | find('id', 2) &#125;&#125;` |
| `pluck(path)` | Returns an array with values resolved from a dot path for each item. | `&#123;&#123; flowContext.users | pluck('profile.name') &#125;&#125;` |
| `includes(search)` | Returns `true` if an array contains `search` or a string contains `search`. | `&#123;&#123; flowContext.tags | includes('sports') &#125;&#125;` |
| `unique` | Returns a de-duplicated array, preserving first-seen order. | `&#123;&#123; flowContext.terms | unique &#125;&#125;` |
| `flatten` | Recursively flattens nested arrays into a single-level array. | `&#123;&#123; flowContext.nestedItems | flatten &#125;&#125;` |
| `chunk(size)` | Splits an array into chunks of `size` (invalid size returns empty array). | `&#123;&#123; flowContext.items | chunk(3) &#125;&#125;` |
| `keys` | Returns own enumerable property names. | `&#123;&#123; flowContext.user | keys &#125;&#125;` |
| `values` | Returns own enumerable property values. | `&#123;&#123; flowContext.user | values &#125;&#125;` |
| `entries` | Returns own enumerable key-value pairs. | `&#123;&#123; flowContext.user | entries &#125;&#125;` |
| `pick(...keys)` | Returns a new object with only the requested own properties. | `&#123;&#123; flowContext.user | pick('id', 'name') &#125;&#125;` |
| `isString` | Returns `true` when `value` is a string. | `&#123;&#123; flowContext.title | isString &#125;&#125;` |
| `isNumber` | Returns `true` when `value` is a number. | `&#123;&#123; flowContext.score | isNumber &#125;&#125;` |
| `isObject` | Returns `true` when `value` is a non-null object (arrays included). | `&#123;&#123; flowContext.payload | isObject &#125;&#125;` |
| `isNull` | Returns `true` only when `value === null`. | `&#123;&#123; flowContext.optionalField | isNull &#125;&#125;` |
| `contains(fragment)` | Returns `true` when `text` contains `fragment`. | `&#123;&#123; flowContext.title | contains('Breaking') &#125;&#125;` |
| `startsWith(prefix)` | Returns `true` when `text` starts with `prefix`. | `&#123;&#123; flowContext.slug | startsWith('news-') &#125;&#125;` |
| `endsWith(suffix)` | Returns `true` when `text` ends with `suffix`. | `&#123;&#123; flowContext.filename | endsWith('.xml') &#125;&#125;` |
| `split(separator, limit?)` | Splits text by separator (string or regex). | `&#123;&#123; flowContext.csv | split(',') &#125;&#125;` |
| `padStart(length, pad?)` | Left-pads text to the target length. | `&#123;&#123; flowContext.sequence | padStart(4, '0') &#125;&#125;` |
| `match(pattern, flags?)` | Returns regex match result or `null` for invalid input/pattern. | `&#123;&#123; flowContext.text | match('\\d+', 'g') &#125;&#125;` |
| `json` | Returns `JSON.stringify(value)` output, or empty string if serialization fails. | `&#123;&#123; flowContext.payload | json &#125;&#125;` |
| `toInt` | Converts to integer with fallback `0` when conversion fails. | `&#123;&#123; flowContext.countText | toInt &#125;&#125;` |
| `toFloat` | Converts to float with fallback `0` when conversion fails. | `&#123;&#123; flowContext.scoreText | toFloat &#125;&#125;` |

### Client Functions

Access Swing/Prime client methods:

```yaml
- step: SET
  # Get metadata
  title: "{{ client.getMetadata().metadata.general.title }}"

  # Get document info
  workFolder: "{{ client.getDocumentWorkfolder() }}"

  # Get selected element
  element: "{{ client.getSelectedElement() }}"

  # Get text content with options
  mainText: '{{ client.getTextContent({ excludeTags: ["caption"] }) | safe }}'
```

## ## Best Practices

### 1. Use Descriptive Variable Names

```yaml
# Good
- step: SET
  selectedArticleText: "{{ client.getSelectedText() }}"

# Avoid
- step: SET
  txt: "{{ client.getSelectedText() }}"
```

### 2. Check Variable Existence

```yaml
- step: IF
  condition: "{{ flowContext.myText }}"
  then:
    - step: HUB_COMPLETION
      instruction: "Process: {{ flowContext.myText }}"
  else:
    - step: SHOW_NOTIFICATION
      message: "Please select text first"
```

### 3. Use Correct Syntax for Variable Type

```yaml
# Built-in variables - single braces
- step: HUB_COMPLETION
  instruction: "Process: {textContent}"

# Custom variables - double braces with flowContext
- step: SHOW_NOTIFICATION
  message: "Result: {{ flowContext.myCustomVariable }}"
```

### 4. Avoid Overwriting Built-in Variables

```yaml
# Be careful not to overwrite built-in variables
- step: SET
  textContent: "My custom text"  # This could cause confusion!
```

## Debugging Variables

Use the `DEBUG` step to inspect the flow context:

```yaml
- step: DEBUG
  # This will log all variables to the browser console
```

Or show specific variables:

```yaml
- step: SHOW_NOTIFICATION
  message: "Debug: myVar = {{ flowContext.myVar }}"
```

## Common Patterns

### Pattern 1: Conditional Processing

```yaml
- step: IF
  condition: "{{ client.isTextSelected() }}"
  then:
    - step: HUB_COMPLETION
      instruction: "Improve: {selectedText}"
    - step: REPLACE_TEXT
      at: CURSOR
  else:
    - step: SHOW_NOTIFICATION
      message: "Please select text first"
```

### Pattern 2: Multi-step Processing

```yaml
- step: HUB_COMPLETION
  instruction: "Generate article ideas about {selectedText}"
  response_format: "list"

- step: USER_SELECT
  promptText: "Choose an idea:"

- step: HUB_COMPLETION
  instruction: "Write a full article about the selected topic"

- step: INSERT_TEXT
  at: CURSOR 
```

## Next Steps

- Learn about [Using Services](./using-services.md) to work with external APIs
- Review the [Step Library](../step-library/overview.md) to see all variable options
- Check out [Configuration Basics](../guides/configuration/basics.md) for advanced patterns
