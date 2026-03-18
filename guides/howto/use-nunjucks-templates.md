# How to Use Nunjucks Templates and Client Functions

ProActions uses Nunjucks templating to provide dynamic variable access and transformations. This guide explains how to use template syntax and access client functions for advanced workflows.

## Template Syntax Overview

ProActions supports two variable syntaxes:

### Built-in Variables (Single Braces)

Use single braces `{}` for built-in ProActions variables:

```yaml
- step: HUB_COMPLETION
  instruction: "Process: {textContent}"

- step: SHOW_NOTIFICATION
  message: "User: {userName}"
```

### Template Expressions (Double Braces)

Use double braces `{{ }}` for:
- Custom flow variables
- Client function calls
- Filters and transformations

```yaml
- step: SET
  myVariable: "Hello"

- step: SHOW_NOTIFICATION
  message: "{{ flowContext.myVariable }}"
```

## Accessing Flow Context

All variables are stored in `flowContext`:

```yaml
- step: SET
  articleTitle: "Breaking News"
  wordCount: 500
  isPublished: false

- step: SHOW_NOTIFICATION
  message: |
    Title: {{ flowContext.articleTitle }}
    Words: {{ flowContext.wordCount }}
    Published: {{ flowContext.isPublished }}
```

### Nested Object Access

Use dot notation to access nested properties:

```yaml
- step: SET
  article:
    title: "My Article"
    author:
      name: "John Doe"
      email: "john@example.com"
    tags:
      - news
      - technology

- step: SHOW_NOTIFICATION
  message: |
    Title: {{ flowContext.article.title }}
    Author: {{ flowContext.article.author.name }}
    Email: {{ flowContext.article.author.email }}
    First Tag: {{ flowContext.article.tags[0] }}
```

## Client Functions

The `client` object provides access to document and platform functions.

### Document Information

```yaml
- step: SET
  # Document identifiers
  docId: "{{ client.getDocumentId() }}"
  docLoid: "{{ client.getDocumentLoid() }}"
  docUuid: "{{ client.getDocumentUuid() }}"
  docName: "{{ client.getDocumentName() }}"

  # Document properties
  objectType: "{{ client.getObjectType() }}"
  workflowStatus: "{{ client.getWorkflowStatus() }}"
  productName: "{{ client.getProductName() }}"
  channel: "{{ client.getChannel() }}"
  issueDate: "{{ client.getDocumentIssueDate() }}"
```

### Metadata Access

```yaml
- step: SET
  # Get entire user metadata object
  metadata: "{{ client.getMetadata() }}"

  # Access specific metadata fields (depends on your Méthode configuration)
  title: "{{ client.getMetadata().metadata.general.title }}"
  author: "{{ client.getMetadata().metadata.general.author }}"
  category: "{{ client.getMetadata().metadata.general.category }}"

  # Get metadata as string
  metadataString: "{{ client.getMetadataString() | safe }}"
```

### System Metadata

```yaml
- step: SET
  systemMetadata: "{{ client.getSystemMetadata().props.productInfo.issueDate }}"
  systemMetadataString: "{{ client.getSystemMetadataString() | safe }}"
```

### Virtual Attributes

```yaml
- step: SET
  virtualAttributes: "{{ client.getVirtualAttributes() }}"
  virtualAttributesString: "{{ client.getVirtualAttributesString() | safe }}"
```

### Document Workfolder

```yaml
- step: SET
  workFolder: "{{ client.getDocumentWorkfolder() }}"
```

### Selected Element Information

```yaml
- step: SET
  selectedElement: "{{ client.getSelectedElement() }}"
  selectedElementXpath: "{{ client.getSelectedElementXpath() }}"
```

### Content Retrieval

```yaml
- step: SET
  # Get text content with options
  text: '{{ client.getTextContent() | safe }}'

  # Get text excluding certain tags
  textNoCaption: '{{ client.getTextContent({ excludeTags: ["caption", "credit"] }) | safe }}'

  # Get XML content
  xml: '{{ client.getXmlContent() | safe }}'
```

### Container Information (for DWPs and Reports)

```yaml
- step: SET
  # Container identifiers
  containerId: "{{ client.getContainerId() }}"
  containerLoid: "{{ client.getContainerLoid() }}"
  containerUuid: "{{ client.getContainerUuid() }}"

  # Container properties
  containerObjectType: "{{ client.getContainerObjectType() }}"
  containerWorkflowStatus: "{{ client.getContainerWorkflowStatus() }}"
  containerProductName: "{{ client.getContainerProductName() }}"
  containerChannel: "{{ client.getContainerChannel() }}"

  # Container metadata
  containerMetadata: "{{ client.getContainerMetadata() }}"
```

### Platform Information

```yaml
- step: SET
  appName: "{{ client.getAppName() }}"  # 'swing' or 'prime'
  baseUrl: "{{ client.getBaseUrl() }}"
  isReadonly: "{{ client.isReadonly() }}"
```

## Nunjucks Filters

Filters transform values using the pipe `|` operator.

### Safe Filter

Prevents HTML escaping (use for HTML/XML content):

```yaml
- step: SET
  htmlContent: '{{ client.getTextContent() | safe }}'
```

### String Filters

```yaml
- step: SET
  # Uppercase
  upper: "{{ flowContext.text | upper }}"

  # Lowercase
  lower: "{{ flowContext.text | lower }}"

  # Capitalize first letter
  capitalize: "{{ flowContext.text | capitalize }}"

  # Title case
  title: "{{ flowContext.text | title }}"

  # Trim whitespace
  trim: "{{ flowContext.text | trim }}"

  # Truncate with length
  short: "{{ flowContext.longText | truncate(50) }}"

  # Truncate with custom ending
  shortCustom: "{{ flowContext.longText | truncate(50, true, '...') }}"
```

### Regex Replace

```yaml
- step: SET
  # Remove invalid filename characters
  filename: "{{ flowContext.title | replace(r/[\\/:*?\"<>|]/g, '_') }}"

  # Remove multiple spaces
  cleaned: "{{ flowContext.text | replace(r/\\s+/g, ' ') }}"

  # Remove special characters
  alphanumeric: "{{ flowContext.text | replace(r/[^a-zA-Z0-9 ]/g, '') }}"
```

### Array Filters

```yaml
- step: SET
  # First item
  first: "{{ flowContext.myList | first }}"

  # Last item
  last: "{{ flowContext.myList | last }}"

  # Join array into string
  joined: "{{ flowContext.tags | join(', ') }}"

  # Array length
  count: "{{ flowContext.myList | length }}"
```

### Custom Resolver Filters

ProActions also adds custom filters for object and array traversal. These are available in the same `{{ }}` templates and can be chained with built-in Nunjucks filters.

```yaml
- step: SET
  users:
    - id: 1
      profile:
        name: "Ada"
        role: "Editor"
    - id: 2
      profile:
        name: "Linus"
        role: "Reviewer"

  # Find one item, then safely access nested attributes
  reviewerName: "{{ flowContext.users | find('id', 2) | getAttr('profile') | getAttr('name') | default('Unknown') }}"

  # Extract one attribute path from each item
  userNames: "{{ flowContext.users | pluck('profile.name') | join(', ') }}"
```

Notes:

- `find(attribute, value)` returns the first matching array item.
- `getAttr(attribute)` reads own properties only (no prototype traversal).
- `pluck(path)` returns an array of values for each element using a dot path.

### Number Filters

```yaml
- step: SET
  # Round number
  rounded: "{{ flowContext.decimal | round }}"

  # Round to decimals
  precise: "{{ flowContext.decimal | round(2) }}"

  # Absolute value
  absolute: "{{ flowContext.number | abs }}"
```

### Default Filter

Provide default value if variable is undefined:

```yaml
- step: SET
  value: "{{ flowContext.maybeUndefined | default('Default Value') }}"
```

`default(...)` keeps the standard built-in Nunjucks behavior. The custom resolver filters do not override it.

## Conditional Expressions

### If-Else in Templates

```yaml
- step: SET
  message: "{{ 'Published' if flowContext.isPublished else 'Draft' }}"
```

### Ternary Operations

```yaml
- step: SET
  status: "{{ flowContext.count > 100 and 'Long' or 'Short' }}"
```

## Combining Client Functions and Filters

### Example: Clean Filename from Metadata

```yaml
- step: SET
  # Get title from metadata and create safe filename
  filename: "{{ client.getMetadata().metadata.general.title | replace(r/[\\/:*?\"<>|]/g, '_') | truncate(50) }}.txt"
```

### Example: Formatted Text Content

```yaml
- step: SET
  # Get text without captions, trim, and truncate
  summary: '{{ client.getTextContent({ excludeTags: ["caption", "credit"] }) | trim | truncate(200) }}'
```

### Example: Document Info String

```yaml
- step: SET
  docInfo: "Document: {{ client.getDocumentName() }} (ID: {{ client.getDocumentId() }}) - Status: {{ client.getWorkflowStatus() | upper }}"
```

## Practical Examples

### Example 1: Generate Filename from Metadata

```yaml
- title: 'Export with Custom Filename'
  flow:
    - step: SET
      # Create filename from title and date
      exportFilename: "{{ client.getMetadata().metadata.general.title | replace(r/[\\/:*?\"<>|]/g, '_') }}_{{ client.getDocumentIssueDate() }}.txt"

      # Get content
      text: '{{ client.getTextContent() | safe }}'

    - step: DOWNLOAD
      filename: "{{ flowContext.exportFilename }}"
```

### Example 2: Text-to-Speech with Metadata

```yaml
- title: 'Create Audio Version'
  flow:
    - step: SET
      # Get text excluding images and captions
      text: '{{ client.getTextContent({ excludeTags: ["web-image-caption", "caption", "credit"] }) | safe }}'

      # Create filename from metadata
      targetName: "{{ client.getMetadata().metadata.general.title | replace(r/[\\/:?]/g, '') | truncate(50) }}.mp3"

    - step: HUB_SPEECH

    - step: UPLOAD
      options:
        name: "{{ flowContext.targetName | safe }}"
        workFolder: "{{ client.getDocumentWorkfolder() }}"
        folderPath: "/Production/Product/Audio"
        type: "Audio"
```

### Example 3: Conditional Processing Based on Metadata

```yaml
- title: 'Process Based on Category'
  flow:
    - step: SET
      category: "{{ client.getMetadata().metadata.general.category }}"

    - step: IF
      condition: "{{ flowContext.category === 'Breaking News' }} "
      then:
        - step: HUB_COMPLETION
          behavior: "Generate urgent, attention-grabbing content."
          instruction: "Create breaking news alert: {textContent}"
      else:
        - step: HUB_COMPLETION
          behavior: "Generate standard professional content."
          instruction: "Summarize: {textContent}"
```

### Example 4: Multi-Document Processing

```yaml
- title: 'Process Container Documents'
  flow:
    - step: SET
      containerName: "{{ client.getContainerMetadata().metadata.general.title }}"
      documentCount: 0

    - step: SHOW_NOTIFICATION
      message: "Processing container: {{ flowContext.containerName }}"
```

### Example 5: Smart Content Export

```yaml
- title: 'Smart Export'
  flow:
    - step: SET
      # Build comprehensive document info
      docType: "{{ client.getObjectType() }}"
      docTitle: "{{ client.getMetadata().metadata.general.title | trim }}"
      docAuthor: "{{ client.getMetadata().metadata.general.author | default('Unknown') }}"
      docDate: "{{ client.getDocumentIssueDate() }}"

      # Create structured filename
      filename: "{{ flowContext.docType }}_{{ flowContext.docTitle | replace(r/[\\s]+/g, '_') }}_{{ flowContext.docDate }}.txt"

      # Get content
      content: '{{ client.getTextContent() | safe }}'

      # Build export text
      text: |-
        Document: {{ flowContext.docTitle }}
        Type: {{ flowContext.docType }}
        Author: {{ flowContext.docAuthor }}
        Date: {{ flowContext.docDate }}

        ---

        {{ flowContext.content }}

    - step: DOWNLOAD
      filename: "{{ flowContext.filename }}"
```

## Advanced Techniques

### Accessing Nested Metadata Safely

Use default filter to handle missing fields:

```yaml
- step: SET
  title: "{{ client.getMetadata().metadata.general.title | default('Untitled') }}"
  subtitle: "{{ client.getMetadata().metadata.general.subtitle | default('') }}"
```

### Building Complex Paths

```yaml
- step: SET
  workFolder: "{{ client.getDocumentWorkfolder() }}"
  category: "{{ client.getMetadata().metadata.general.category | default('General') }}"
  year: "{{ client.getDocumentIssueDate() | truncate(4, true, '') }}"

  targetPath: "{{ flowContext.workFolder }}/{{ flowContext.category }}/{{ flowContext.year }}"
```

### Dynamic Service Selection

```yaml
- step: SET
  language: "{{ client.getMetadata().metadata.general.language | default('EN') }}"

  targetLang: "{{ 'FR' if flowContext.language === 'EN' else 'EN' }}"

- step: DEEPL_TRANSLATE
  instruction: "{selectedText}"
  target_lang: "{{ flowContext.targetLang }}"
```

## Debugging Templates

### Check Template Output

```yaml
- step: SET
  testValue: "{{ client.getMetadata().metadata.general.title }}"

- step: DEBUG

- step: SHOW_NOTIFICATION
  message: "Value: {{ flowContext.testValue }}"
```

### Use SCRIPTING for Complex Logic

When templates get too complex, use SCRIPTING:

```yaml
- step: SCRIPTING
  script: |
    const metadata = client.getMetadata();
    const title = metadata?.metadata?.general?.title || 'Untitled';
    const date = client.getDocumentIssueDate();
    const filename = `${title.replace(/[\\/:*?"<>|]/g, '_')}_${date}.txt`;
    return { filename, title, date };
```

## Best Practices

### 1. Use Safe Filter for Content

```yaml
# Good - prevents HTML escaping
text: '{{ client.getTextContent() | safe }}'

# Avoid - may escape HTML entities
text: "{{ client.getTextContent() }}"
```

### 2. Provide Defaults

```yaml
# Good - handles missing metadata
title: "{{ client.getMetadata().metadata.general.title | default('Untitled') }}"

# Risky - may fail if metadata missing
title: "{{ client.getMetadata().metadata.general.title }}"
```

### 3. Clean User-Generated Content

```yaml
# Good - removes invalid characters
filename: "{{ flowContext.userInput | replace(r/[\\/:*?\"<>|]/g, '_') }}"

# Risky - may contain invalid characters
filename: "{{ flowContext.userInput }}"
```

### 4. Keep Templates Readable

```yaml
# Good - split complex operations
- step: SET
  rawTitle: "{{ client.getMetadata().metadata.general.title }}"
  cleanTitle: "{{ flowContext.rawTitle | replace(r/[\\s]+/g, '_') }}"
  filename: "{{ flowContext.cleanTitle }}.txt"

# Avoid - hard to read
- step: SET
  filename: "{{ client.getMetadata().metadata.general.title | replace(r/[\\s]+/g, '_') }}.txt"
```

### 5. Document Complex Templates

```yaml
- step: SET
  # Extract title from metadata and create safe filename:
  # 1. Get title from metadata
  # 2. Replace spaces with underscores
  # 3. Remove invalid filename characters
  # 4. Truncate to 50 characters
  # 5. Add extension
  filename: "{{ client.getMetadata().metadata.general.title | replace(r/[\\s]+/g, '_') | replace(r/[\\/:*?\"<>|]/g, '') | truncate(50, true, '') }}.txt"
```

## Next Steps

- **[Working with Variables](../../getting-started/working-with-variables.md)** - Variable fundamentals
- **[Configuration Basics](../configuration/basics.md)** - Configuration overview
- **[Debug Flows](./debug-flows.md)** - Debugging workflows
