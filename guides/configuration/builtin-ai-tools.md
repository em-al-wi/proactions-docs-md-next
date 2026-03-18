---
sidebar_position: 7
---

# Built-in AI Tools Reference

Complete reference for all built-in AI tools available in ProActions.

## Overview

ProActions provides pre-configured tools that can be easily enabled for AI function calling. 
These tools provide direct access to document manipulation, user interaction, and metadata without 
requiring you to write custom function definitions.

## Configuration

Enable tools in your workflow step configuration:

```yaml
- step: HUB_COMPLETION
  instruction: "Analyze this document"
  builtinTools:
    - ContentReadTools      # Enable all tools in this class
    - DocumentInfoTools     # Enable all tools in this class
```

You can also be more specific:

```yaml
- step: HUB_COMPLETION
  instruction: "Analyze this document"
  builtinTools:
    - class: ContentReadTools
      only: [getTextContent, getTextAtXpath]  # Only enable these tools
      xpaths:
        headline: "/doc/story/headline"       # Add xpath aliases
        body: "/doc/story/text"
```

Tools are exposed to the model using these name formats:

- `<toolName>` for built-in ClientAdapter tools
- `<toolClass>__<toolName>` for provider tools
- `mcp__<serverName>__<toolName>` for MCP tools
- `<aliasName>` for aliases (no prefix unless there’s a collision, then `alias__` is used)

### Tool Aliases

You can define custom virtual tools with preset arguments:

```yaml
builtinTools:
  - alias: getHeadline
    target: getTextAtXpath
    description: "Returns the main headline"
    args:
      xpath: "/doc/story/headline"
```

You can also alias MCP tools:

```yaml
builtinTools:
  - alias: saveReport
    target: mcp__filesystem__write_file
    description: "Save the generated report"
    args:
      path: "/documents/report.md"
```

Alias targets support shorthand formats:

- `<tool>` (built-in ClientAdapter tools)
- `<toolClass>__<tool>` or `<toolClass>.<tool>` (provider tools)
- `mcp__<server>__<tool>`, `mcp.<server>.<tool>`, or `<server>.<tool>` (MCP tools)

### Tool Alias Templates

If you need multiple similar aliases, define a template once and extend it:

```yaml
builtinTools:
  - aliasTemplate: markSegments
    target: highlightTextSegments
    description: "Mark segments with consistent matching behavior"
    args:
      tag: "mark"
      virtual: true
      match:
        normalizeCharacters: true
        normalizeWhitespace: true
        caseSensitive: false
        mode: "first"

  - alias: markClaims
    extend: markSegments
    description: "Mark claims for fact checking"
    args:
      attributes:
        class: "fact-check"
```

Alias argument behavior:

- `args` defines fixed top-level defaults for the alias.
- Fixed top-level arguments are removed from the alias schema exposed to the LLM.
- Non-fixed parameters remain available for agent input.
- Runtime arguments are merged with fixed defaults, where fixed values win.
- With `toolStrict: true`, overriding fixed top-level arguments is rejected.


## ContentTools

Tools for reading and writing document content

### Tools


#### `getTextContent`

Get the text content of the current document


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `options` | `object` | No | Options for text extraction (optional) |



**Returns:** `void`


#### `getXmlContent`

Get the XML content of the current document


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `options` | `object` | No | Options for XML extraction (optional) |



**Returns:** `void`


#### `getSelectedText`

Get the currently selected text in the editor


**Parameters:** None


**Returns:** `void`


#### `getSelectedXML`

Get the XML of the currently selected content


**Parameters:** None


**Returns:** `void`


#### `getParagraphText`

Get the text of the current paragraph where the cursor is located


**Parameters:** None


**Returns:** `void`


#### `getParagraphXML`

Get the XML of the current paragraph where the cursor is located


**Parameters:** None


**Returns:** `void`


#### `getXmlAtXpath`

Get XML content at a specific XPath location


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `xpath` | `string` | Yes | XPath expression to locate the content |



**Returns:** `void`


#### `getTextAtXpath`

Get text content at a specific XPath location


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `xpath` | `string` | Yes | XPath expression to locate the content |



**Returns:** `void`


#### `isTextSelected`

Check if any text is currently selected in the editor


**Parameters:** None


**Returns:** `void`


#### `getSelectedElement`

Get the name of the currently selected XML element


**Parameters:** None


**Returns:** `void`


#### `getSelectedElementXpath`

Get the XPath of the currently selected element


**Parameters:** None


**Returns:** `void`


#### `getContainerTextContent`

Get text content from the container (for container documents)


**Parameters:** None


**Returns:** `void`


#### `getContainerXmlContents`

Get XML contents from all items in the container


**Parameters:** None


**Returns:** `void`


#### `insertTextAtCursor`

Insert text at the current cursor position


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `text` | `string` | Yes | Text to insert |



**Returns:** `void`


#### `insertTextAtXpath`

Insert text at a specific XPath location


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `text` | `string` | Yes | Text to insert |

| `xpath` | `string` | Yes | XPath expression for insertion location |



**Returns:** `void`


#### `insertXmlAtCursor`

Insert XML at the current cursor position


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `xml` | `string` | Yes | XML content to insert |

| `insertType` | `string` | No | How to insert the XML (insertInline, insertBefore, insertAfter) |



**Returns:** `void`


#### `insertXmlAtXpath`

Insert XML at a specific XPath location


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `xml` | `string` | Yes | XML content to insert |

| `xpath` | `string` | Yes | XPath expression for insertion location |



**Returns:** `void`


#### `replaceTextAtCursor`

Replace the selected text at cursor with new text


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `text` | `string` | Yes | New text to replace selection |



**Returns:** `void`


#### `replaceTextAtCursorParagraph`

Replace the entire paragraph at cursor with new text


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `text` | `string` | Yes | New text to replace paragraph |



**Returns:** `void`


#### `replaceTextAtXpath`

Replace text at a specific XPath location


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `text` | `string` | Yes | New text |

| `xpath` | `string` | Yes | XPath expression for replacement location |



**Returns:** `void`


#### `replaceXmlAtCursor`

Replace the selected XML at cursor with new XML


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `xml` | `string` | Yes | New XML to replace selection |



**Returns:** `void`


#### `replaceXmlAtCursorParagraph`

Replace the entire paragraph XML at cursor with new XML


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `xml` | `string` | Yes | New XML to replace paragraph |



**Returns:** `void`


#### `replaceXmlAtXpath`

Replace XML at a specific XPath location


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `xml` | `string` | Yes | New XML content |

| `xpath` | `string` | Yes | XPath expression for replacement location |



**Returns:** `void`



## ContainerInfoTools

Tools for retrieving information about the container (for container documents)

### Tools


#### `getContainerId`

Get the ID of the container


**Parameters:** None


**Returns:** `void`


#### `getContainerLoid`

Get the LOID of the container


**Parameters:** None


**Returns:** `void`


#### `getContainerUuid`

Get the UUID of the container


**Parameters:** None


**Returns:** `void`


#### `getContainerObjectType`

Get the object type of the container


**Parameters:** None


**Returns:** `void`


#### `getContainerWorkflowStatus`

Get the workflow status of the container


**Parameters:** None


**Returns:** `void`


#### `getContainerProductName`

Get the product name of the container


**Parameters:** None


**Returns:** `void`


#### `getContainerChannel`

Get the channel of the container


**Parameters:** None


**Returns:** `void`


#### `getContainerMetadata`

Get the metadata of the container as an object


**Parameters:** None


**Returns:** `void`


#### `getContainerMetadataString`

Get the metadata of the container as a string


**Parameters:** None


**Returns:** `void`


#### `getContainerSystemMetadata`

Get the system metadata of the container as an object


**Parameters:** None


**Returns:** `void`


#### `getContainerSystemMetadataString`

Get the system metadata of the container as a string


**Parameters:** None


**Returns:** `void`


#### `getContainerVirtualAttributes`

Get the virtual attributes of the container as an object


**Parameters:** None


**Returns:** `void`


#### `getContainerVirtualAttributesString`

Get the virtual attributes of the container as a string


**Parameters:** None


**Returns:** `void`


#### `getContainerUsageTickets`

Get the usage tickets of the container as an object


**Parameters:** None


**Returns:** `void`


#### `getContainerUsageTicketsString`

Get the usage tickets of the container as a string


**Parameters:** None


**Returns:** `void`



## CustomTools

Custom tool provider used for validation

### Tools


#### `getCurrentTimestamp`

Get the current timestamp in ISO-8601 format.


**Parameters:** None


**Returns:** `ISO-8601 timestamp string.`


#### `getDocumentSummaryInfo`

Return summary information about the current document.


**Parameters:** None


**Returns:** `Document identifiers and workflow metadata.`



## DocumentInfoTools

Tools for retrieving information about the current document

### Tools


#### `getDocumentId`

Get the ID of the current document


**Parameters:** None


**Returns:** `void`


#### `getDocumentLoid`

Get the LOID (Logical Object ID) of the current document


**Parameters:** None


**Returns:** `void`


#### `getDocumentUuid`

Get the UUID of the current document


**Parameters:** None


**Returns:** `void`


#### `getDocumentName`

Get the name of the current document


**Parameters:** None


**Returns:** `void`


#### `getDocumentIssueDate`

Get the issue date of the current document


**Parameters:** None


**Returns:** `void`


#### `getObjectType`

Get the object type of the current document


**Parameters:** None


**Returns:** `void`


#### `getWorkflowStatus`

Get the workflow status of the current document


**Parameters:** None


**Returns:** `void`


#### `getProductName`

Get the product name of the current document


**Parameters:** None


**Returns:** `void`


#### `getChannel`

Get the channel of the current document


**Parameters:** None


**Returns:** `void`


#### `getMetadata`

Get the metadata of the current document as an object


**Parameters:** None


**Returns:** `void`


#### `getMetadataString`

Get the metadata of the current document as a string


**Parameters:** None


**Returns:** `void`


#### `getSystemMetadata`

Get the system metadata of the current document as an object


**Parameters:** None


**Returns:** `void`


#### `getSystemMetadataString`

Get the system metadata of the current document as a string


**Parameters:** None


**Returns:** `void`


#### `getVirtualAttributes`

Get the virtual attributes of the current document as an object


**Parameters:** None


**Returns:** `void`


#### `getVirtualAttributesString`

Get the virtual attributes of the current document as a string


**Parameters:** None


**Returns:** `void`


#### `getUsageTickets`

Get the usage tickets of the current document as an object


**Parameters:** None


**Returns:** `void`


#### `getUsageTicketsString`

Get the usage tickets of the current document as a string


**Parameters:** None


**Returns:** `void`


#### `isReadonly`

Check if the current document is read-only


**Parameters:** None


**Returns:** `void`


#### `getDocumentWorkfolder`

Get the workfolder of the current document


**Parameters:** None


**Returns:** `void`



## FeedbackTools

Tools for providing feedback and progress updates to the user

### Tools


#### `addFeedback`

Add a feedback message to the execution monitor


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `type` | `string` | Yes | Type of feedback |

| `content` | `string` | Yes | Feedback message content |

| `metadata` | `object` | No | Optional metadata for the feedback |



**Returns:** `void`


#### `updateProgress`

Update progress with a percentage (0-100)


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `percentage` | `number` | Yes | Completion percentage (0-100) |

| `message` | `string` | No | Optional status message |



**Returns:** `void`


#### `addTableFeedback`

Display data as a table in the monitor. Example rows: `[{"Metric":"Readability","Score":72},{"Metric":"Length","Score":54}]`


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `data` | `array` | Yes | Array of row objects. Each object represents one row where keys are column names and values are cell values (string/number/boolean). |

| `title` | `string` | No | Optional table title |

| `options` | `object` | No | Display options (e.g., maxRows) |



**Returns:** `void`


#### `addJsonFeedback`

Display data as formatted JSON in the monitor


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `data` | `object` | Yes | Data to display as JSON (object or array) |

| `title` | `string` | No | Optional title |

| `options` | `object` | No | Display options (e.g., maxDepth) |



**Returns:** `void`


#### `addKeyValueFeedback`

Display key-value pairs in the monitor


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `data` | `object` | Yes | Key-value pairs to display |

| `title` | `string` | No | Optional title |



**Returns:** `void`


#### `addMetricsFeedback`

Display metrics (numbers with labels) in the monitor. Provide metrics as a key-value object.


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `metrics` | `object` | Yes | Metrics to display as key-value pairs (object: `{label: value}`) |

| `title` | `string` | No | Optional title |



**Returns:** `void`


#### `showNotification`

Show a notification to the user


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `type` | `string` | Yes | Notification type |

| `message` | `string` | Yes | Notification message |

| `title` | `string` | No | Optional notification title |



**Returns:** `void`


#### `highlightTextSegments`

Highlight text segments by matching their content in the document, then adding pure text tags.


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `segments` | `array` | Yes | Segments to highlight |

| `virtual` | `boolean` | No | Virtual tags (Swing only) |

| `tag` | `string` | No | Override tag to apply to all segments |

| `match` | `object` | No | Override match options to apply to all segments |

| `attributes` | `object` | No | Override attributes to apply to all segments |



**Returns:** `void`



## HighlightTools

Tools for marking and highlighting text content

### Tools


#### `getPureText`

Get the pure text content of the document or report without any formatting or markup


**Parameters:** None


**Returns:** `void`


#### `addPureTextTags`

Add tags to pure text content


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `tags` | `array` | Yes | Array of tags to add |

| `virtual` | `boolean` | No | Whether tags are virtual and will not be persisted(optional) |



**Returns:** `void`


#### `clearPureTextTags`

Clear specific tags from from a document or report


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `tags` | `array` | Yes | Array of tags to be removed |



**Returns:** `void`



## HubMcp

No description provided

### Tools


#### `callMcpTool`

Execute a tool on an MCP server via the Hub


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `server` | `string` | Yes | The MCP server name |

| `tool` | `string` | Yes | The tool name |

| `arguments` | `object` | Yes | Tool arguments |



**Returns:** `void`



## UserInfoTools

Tools for retrieving information about the current user

### Tools


#### `getUserName`

Get the username of the current user


**Parameters:** None


**Returns:** `void`


#### `getUserFullName`

Get the full name of the current user


**Parameters:** None


**Returns:** `void`


#### `getUserGroups`

Get the groups the current user belongs to


**Parameters:** None


**Returns:** `void`


#### `getUserTeams`

Get the teams the current user belongs to


**Parameters:** None


**Returns:** `void`


#### `getUserMetadata`

Get the metadata of the current user as an object


**Parameters:** None


**Returns:** `void`


#### `getUserMetadataString`

Get the metadata of the current user as a string


**Parameters:** None


**Returns:** `void`


#### `getUserSystemMetadata`

Get the system metadata of the current user as an object


**Parameters:** None


**Returns:** `void`


#### `getUserSystemMetadataString`

Get the system metadata of the current user as a string


**Parameters:** None


**Returns:** `void`



## UserInteractionTools

Tools for prompting user for input and decisions

### Tools


#### `askSingleChoice`

Ask the user to select ONE answer from a list of choices. User sees the question and answer buttons in the monitor. Optionally supports custom text input and skip functionality.


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `question` | `string` | Yes | The question to ask the user |

| `answers` | `array` | Yes | Array of answer options |

| `allowCustomInput` | `boolean` | No | If true, shows a text input field below the buttons for custom user input |

| `allowSkip` | `boolean` | No | If true, automatically adds a "Skip" button to the choices |

| `skipLabel` | `string` | No | Custom label for the skip button (default: "Skip"). Only used when allowSkip is true |



**Returns:** `JSON: { answerId: string, answerLabel: string, customInput?: string }`


#### `askMultipleChoice`

Ask the user to select MULTIPLE answers from a list. User sees checkboxes for each option and a Submit button.


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `question` | `string` | Yes | The question to ask the user |

| `answers` | `array` | Yes | Array of answer options |

| `minSelect` | `integer` | No | Minimum number of selections required (default: 0) |

| `maxSelect` | `integer` | No | Maximum number of selections allowed (default: unlimited) |

| `allowCustomInput` | `boolean` | No | If true, allows optional custom text input below options |



**Returns:** `JSON: { selectedIds: string[], selectedLabels: string[], customInput?: string }`


#### `askFreeformQuestion`

Ask the user to type a freeform text answer. User sees a text input field and Submit button.


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `question` | `string` | Yes | The question to ask the user |

| `placeholder` | `string` | No | Placeholder text for the input field |

| `required` | `boolean` | No | Whether an answer is required (default: true) |



**Returns:** `JSON: { answer: string }`


#### `askConfirmation`

Ask the user a yes/no confirmation question. User sees Yes and No buttons.


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `question` | `string` | Yes | The confirmation question to ask |



**Returns:** `JSON: { confirmed: boolean }`



## UtilityTools

Utility tools for data conversion and manipulation

### Tools


#### `xmlToMarkdown`

Convert XML content to Markdown format


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `xml` | `string` | Yes | XML content to convert |

| `config` | `object` | No | Optional conversion configuration |



**Returns:** `void`


#### `markdownToHtml`

Convert Markdown content to HTML


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `markdown` | `string` | Yes | Markdown content to convert |



**Returns:** `void`


#### `validateAndRepairXml`

Validate and repair malformed XML


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `xml` | `string` | Yes | XML content to validate and repair |



**Returns:** `void`



