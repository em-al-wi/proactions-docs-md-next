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
    - ContentTools          # Enable all tools in this class
    - DocumentInfoTools     # Enable all tools in this class
```

You can also be more specific:

```yaml
- step: HUB_COMPLETION
  instruction: "Analyze this document"
  builtinTools:
    - class: ContentTools
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





#### `getXmlContent`

Get the XML content of the current document


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `options` | `object` | No | Options for XML extraction (optional) |





#### `getSelectedText`

Get the currently selected text in the editor


**Parameters:** None




#### `getSelectedXML`

Get the XML of the currently selected content


**Parameters:** None




#### `getParagraphText`

Get the text of the current paragraph where the cursor is located


**Parameters:** None




#### `getParagraphXML`

Get the XML of the current paragraph where the cursor is located


**Parameters:** None




#### `getXmlAtXpath`

Get XML content at a specific XPath location


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `xpath` | `string` | Yes | XPath expression to locate the content |





#### `getTextAtXpath`

Get text content at a specific XPath location


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `xpath` | `string` | Yes | XPath expression to locate the content |





#### `isTextSelected`

Check if any text is currently selected in the editor


**Parameters:** None




#### `getSelectedElement`

Get the name of the currently selected XML element


**Parameters:** None




#### `getSelectedElementXpath`

Get the XPath of the currently selected element


**Parameters:** None




#### `getContainerTextContent`

Get text content from the container (for container documents)


**Parameters:** None




#### `getContainerXmlContents`

Get XML contents from all items in the container


**Parameters:** None



**Returns:** Array of items, each with the item metadata and its XML string



#### `insertTextAtCursor`

Insert text at the current cursor position


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `text` | `string` | Yes | Text to insert |





#### `insertTextAtXpath`

Insert text at a specific XPath location


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `text` | `string` | Yes | Text to insert |

| `xpath` | `string` | Yes | XPath expression for insertion location |





#### `insertXmlAtCursor`

Insert XML at the current cursor position


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `xml` | `string` | Yes | XML content to insert |

| `insertType` | `string` | No | How to insert the XML (insertInline, insertBefore, insertAfter) |





#### `insertXmlAtXpath`

Insert XML at a specific XPath location


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `xml` | `string` | Yes | XML content to insert |

| `xpath` | `string` | Yes | XPath expression for insertion location |





#### `replaceTextAtCursor`

Replace the selected text at cursor with new text


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `text` | `string` | Yes | New text to replace selection |





#### `replaceTextAtCursorParagraph`

Replace the entire paragraph at cursor with new text


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `text` | `string` | Yes | New text to replace paragraph |





#### `replaceTextAtXpath`

Replace text at a specific XPath location


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `text` | `string` | Yes | New text |

| `xpath` | `string` | Yes | XPath expression for replacement location |





#### `replaceXmlAtCursor`

Replace the selected XML at cursor with new XML


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `xml` | `string` | Yes | New XML to replace selection |





#### `replaceXmlAtCursorParagraph`

Replace the entire paragraph XML at cursor with new XML


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `xml` | `string` | Yes | New XML to replace paragraph |





#### `replaceXmlAtXpath`

Replace XML at a specific XPath location


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `xml` | `string` | Yes | New XML content |

| `xpath` | `string` | Yes | XPath expression for replacement location |






## ContainerInfoTools

Tools for retrieving information about the container (for container documents)

### Tools


#### `getContainerId`

Get the ID of the container


**Parameters:** None




#### `getContainerLoid`

Get the LOID of the container


**Parameters:** None




#### `getContainerUuid`

Get the UUID of the container


**Parameters:** None




#### `getContainerObjectType`

Get the object type of the container


**Parameters:** None




#### `getContainerWorkflowStatus`

Get the workflow status of the container


**Parameters:** None




#### `getContainerProductName`

Get the product name of the container


**Parameters:** None




#### `getContainerChannel`

Get the channel of the container


**Parameters:** None




#### `getContainerMetadata`

Get the metadata of the container as an object


**Parameters:** None




#### `getContainerMetadataString`

Get the metadata of the container as a string


**Parameters:** None




#### `getContainerSystemMetadata`

Get the system metadata of the container as an object


**Parameters:** None




#### `getContainerSystemMetadataString`

Get the system metadata of the container as a string


**Parameters:** None




#### `getContainerVirtualAttributes`

Get the virtual attributes of the container as an object


**Parameters:** None




#### `getContainerVirtualAttributesString`

Get the virtual attributes of the container as a string


**Parameters:** None




#### `getContainerUsageTickets`

Get the usage tickets of the container as an object


**Parameters:** None




#### `getContainerUsageTicketsString`

Get the usage tickets of the container as a string


**Parameters:** None





## CustomTools

Custom tool provider used for validation

### Tools


#### `getCurrentTimestamp`

Get the current timestamp in ISO-8601 format.


**Parameters:** None



**Returns:** ISO-8601 timestamp string.



#### `getDocumentSummaryInfo`

Return summary information about the current document.


**Parameters:** None



**Returns:** Document identifiers and workflow metadata.




## DocumentInfoTools

Tools for retrieving information about the current document

### Tools


#### `getDocumentId`

Get the ID of the current document


**Parameters:** None




#### `getDocumentLoid`

Get the LOID (Logical Object ID) of the current document


**Parameters:** None




#### `getDocumentUuid`

Get the UUID of the current document


**Parameters:** None




#### `getDocumentName`

Get the name of the current document


**Parameters:** None




#### `getDocumentIssueDate`

Get the issue date of the current document


**Parameters:** None




#### `getObjectType`

Get the object type of the current document


**Parameters:** None




#### `getWorkflowStatus`

Get the workflow status of the current document


**Parameters:** None




#### `getProductName`

Get the product name of the current document


**Parameters:** None




#### `getChannel`

Get the channel of the current document


**Parameters:** None




#### `getMetadata`

Get the metadata of the current document as an object


**Parameters:** None




#### `getMetadataString`

Get the metadata of the current document as a string


**Parameters:** None




#### `getSystemMetadata`

Get the system metadata of the current document as an object


**Parameters:** None




#### `getSystemMetadataString`

Get the system metadata of the current document as a string


**Parameters:** None




#### `getVirtualAttributes`

Get the virtual attributes of the current document as an object


**Parameters:** None




#### `getVirtualAttributesString`

Get the virtual attributes of the current document as a string


**Parameters:** None




#### `getUsageTickets`

Get the usage tickets of the current document as an object


**Parameters:** None




#### `getUsageTicketsString`

Get the usage tickets of the current document as a string


**Parameters:** None




#### `isReadonly`

Check if the current document is read-only


**Parameters:** None




#### `getDocumentWorkfolder`

Get the workfolder of the current document


**Parameters:** None





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





#### `updateProgress`

Update progress with a percentage (0-100)


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `percentage` | `number` | Yes | Completion percentage (0-100) |

| `message` | `string` | No | Optional status message |





#### `addTableFeedback`

Display data as a table in the monitor. Example rows: `[{"Metric":"Readability","Score":72},{"Metric":"Length","Score":54}]`


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `data` | `array` | Yes | Array of row objects. Each object represents one row where keys are column names and values are cell values (string/number/boolean). |

| `title` | `string` | No | Optional table title |

| `options` | `object` | No | Display options (e.g., maxRows) |





#### `addJsonFeedback`

Display data as formatted JSON in the monitor


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `data` | `object` | Yes | Data to display as JSON (object or array) |

| `title` | `string` | No | Optional title |

| `options` | `object` | No | Display options (e.g., maxDepth) |





#### `addKeyValueFeedback`

Display key-value pairs in the monitor


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `data` | `object` | Yes | Key-value pairs to display |

| `title` | `string` | No | Optional title |





#### `addMetricsFeedback`

Display metrics (numbers with labels) in the monitor. Provide metrics as a key-value object.


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `metrics` | `object` | Yes | Metrics to display as key-value pairs (object: `{label: value}`) |

| `title` | `string` | No | Optional title |





#### `showNotification`

Show a notification to the user


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `type` | `string` | Yes | Notification type |

| `message` | `string` | Yes | Notification message |

| `title` | `string` | No | Optional notification title |





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






## HighlightTools

Tools for marking and highlighting text content

### Tools


#### `getPureText`

Get the pure text content of the document or report without any formatting or markup


**Parameters:** None




#### `addPureTextTags`

Add tags to pure text content


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `tags` | `array` | Yes | Array of tags to add |

| `virtual` | `boolean` | No | Whether tags are virtual and will not be persisted(optional) |





#### `clearPureTextTags`

Clear specific tags from a document or report


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `tags` | `array` | Yes | Array of tags to be removed |






## HubMcp

Generic passthrough for invoking MCP tools via the Hub when a specific server/tool pair is not pre-configured as an alias.

### Tools


#### `callMcpTool`

Execute a tool on an MCP server via the Hub


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `server` | `string` | Yes | The MCP server name |

| `tool` | `string` | Yes | The tool name |

| `arguments` | `object` | Yes | Tool arguments |




**Returns:** Tool result as JSON string (or raw string if the MCP tool returned a string)




## ToolDiscovery

Meta-tools used by the autoDiscoverTools flow to list available providers and activate them on demand during a completion.

### Tools


#### `listAvailableTools`

List all available tool providers and their tools. Returns a catalog with provider names, descriptions, and tool details. Use this to discover what tools can be activated for the current session.


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `category` | `string` | No | Optional provider name to filter the catalog to a single provider |




**Returns:** JSON catalog of available tool providers and their tools



#### `activateTools`

Activate one or more tool providers to make their tools available for this session. Once activated, the tools become directly callable with full typed parameters. Call listAvailableTools first to see available providers and their tools.


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `providers` | `array` | Yes | Array of provider names to activate (e.g. ["ContentTools", "FeedbackTools"]) |




**Returns:** JSON: \{ activated: string[], toolCount: number, skipped?: string[] \}




## UserInfoTools

Tools for retrieving information about the current user

### Tools


#### `getUserName`

Get the username of the current user


**Parameters:** None




#### `getUserFullName`

Get the full name of the current user


**Parameters:** None




#### `getUserGroups`

Get the groups the current user belongs to


**Parameters:** None




#### `getUserTeams`

Get the teams the current user belongs to


**Parameters:** None




#### `getUserMetadata`

Get the metadata of the current user as an object


**Parameters:** None




#### `getUserMetadataString`

Get the metadata of the current user as a string


**Parameters:** None




#### `getUserSystemMetadata`

Get the system metadata of the current user as an object


**Parameters:** None




#### `getUserSystemMetadataString`

Get the system metadata of the current user as a string


**Parameters:** None





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




**Returns:** JSON: \{ answerId: string, answerLabel: string, customInput?: string \}



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




**Returns:** JSON: \{ selectedIds: string[], selectedLabels: string[], customInput?: string \}



#### `askFreeformQuestion`

Ask the user to type a freeform text answer. User sees a text input field and Submit button.


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `question` | `string` | Yes | The question to ask the user |

| `placeholder` | `string` | No | Placeholder text for the input field |

| `required` | `boolean` | No | Whether an answer is required (default: true) |




**Returns:** JSON: \{ answer: string \}



#### `askConfirmation`

Ask the user a yes/no confirmation question. User sees Yes and No buttons.


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `question` | `string` | Yes | The confirmation question to ask |




**Returns:** JSON: \{ confirmed: boolean \}




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





#### `markdownToHtml`

Convert Markdown content to HTML


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `markdown` | `string` | Yes | Markdown content to convert |





#### `validateAndRepairXml`

Validate and repair malformed XML


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `xml` | `string` | Yes | XML content to validate and repair |






