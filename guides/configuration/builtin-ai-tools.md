---
sidebar_position: 7
---

# Built-in AI Tools Reference

Complete reference for all built-in AI tools available in ProActions.

## Overview

ProActions provides pre-configured tools that can be easily enabled for AI function calling.
These tools provide direct access to document manipulation, user interaction, and metadata without
requiring you to write custom function definitions.

**StoryEditor** is the curated, task-level surface for editorial agents — safe validated text edits, named document fields, and visual marks, with your document's xpaths and safety rules configured once via `config:`. **ContentTools** and **HighlightTools** remain the stable read/write primitives underneath it. These tools work identically in `OPENAI_COMPLETION`/`HUB_COMPLETION` and in voice agents — see the [Voice Agents guide](./voice-agents.md).

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

### Conditional Tools (`when`)

Object entries (`class`, `alias`, and `mcp` forms) support an optional `when` condition — a template expression or boolean evaluated once when the request or voice session is built. When it resolves falsy, the tools are neither advertised to the model nor executable:

```yaml
builtinTools:
  - alias: setHeadline
    target: replaceTextAtXpath
    args:
      xpath: "/doc/story/headline"
    when: "{{ not client.isReadonly() }}"
  - class: ContentTools
    only: [getSelectedText]
    when: "{{ client.isTextSelected() }}"
```

See [Conditional Tool Exposure](./ai-integration.md#conditional-tool-exposure-when) in the AI Integration guide for evaluation semantics.

### Provider Configuration (`config`)

Class entries accept a `config:` block, validated against the provider's declared configuration options (unknown keys warn but never fail). Values support template expressions — resolved once when the request or voice session is built, with access to flowContext, `client.*`, and `global`/`imports` block constants:

```yaml
builtinTools:
  - class: StoryEditor
    only: [searchAndReplaceText, appendParagraph, getField, setField]
    config:
      bodyXpath: "{{ imports.env.doc.body }}"
      allowedElements: [p, h2, h3, ul, li, b, i, u]
      fields:
        headline: "/doc/story/grouphead/headline/p"
        summary: "/doc/story/summary/p"
```

Each provider's options are documented in its **Configuration** section below. See [Configuring Built-in Tools](./ai-integration.md#configuring-built-in-tools-config) in the AI Integration guide for the full semantics.


## ContentTools

Stable read/write primitives for the current document: full-document and selection reads, cursor-anchored paragraph access, and exact-xpath operations. For task-level editorial work (safe edits, named fields, marks) prefer the StoryEditor provider.



### Tools


#### `getTextContent`

Read the full document as plain text. The primary read for answering questions about or quoting from the current document.


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `options` | `object` | No | Options for text extraction (optional) |





#### `getXmlContent`

Read the full document as raw XML including all markup. Use only when document structure matters — for plain content use getTextContent.


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `options` | `object` | No | Options for XML extraction (optional) |





#### `getSelectedText`

Get the text the user has currently selected in the editor. Returns empty when nothing is selected — check isTextSelected first when the selection state is unknown.


**Parameters:** None




#### `getSelectedXML`

Get the raw XML of the user's current selection, including markup. For plain text use getSelectedText.


**Parameters:** None




#### `getParagraphText`

CURSOR-ANCHORED: returns the text of only the paragraph the cursor is in — not the whole document. Use when the user refers to "this paragraph".


**Parameters:** None




#### `getParagraphXML`

CURSOR-ANCHORED: returns the raw XML of only the paragraph the cursor is in. For plain text use getParagraphText.


**Parameters:** None




#### `getXmlAtXpath`

Low-level primitive: read raw XML at an exact document xpath. Requires knowing the document structure.


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `xpath` | `string` | Yes | XPath expression to locate the content |





#### `getTextAtXpath`

Low-level primitive: read plain text at an exact document xpath. Requires knowing the document structure.


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `xpath` | `string` | Yes | XPath expression to locate the content |





#### `isTextSelected`

Check whether the user currently has text selected. Call this before selection-based tools when the selection state is unknown.


**Parameters:** None




#### `getSelectedElement`

Get the name of the currently selected XML element


**Parameters:** None




#### `getSelectedElementXpath`

Get the XPath of the currently selected element


**Parameters:** None




#### `getContainerTextContent`

CONTAINER DOCUMENTS ONLY: read the aggregated text content of the container.


**Parameters:** None




#### `getContainerXmlContents`

CONTAINER DOCUMENTS ONLY: read each contained item with its metadata and XML.


**Parameters:** None



**Returns:** Array of items, each with the item metadata and its XML string



#### `insertTextAtCursor`

CURSOR-ANCHORED write: insert plain text at the current cursor position.


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `text` | `string` | Yes | Text to insert |





#### `insertTextAtXpath`

Low-level primitive: insert plain text at an exact document xpath. Requires knowing the document structure.


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `text` | `string` | Yes | Text to insert |

| `xpath` | `string` | Yes | XPath expression for insertion location |





#### `insertXmlAtCursor`

CURSOR-ANCHORED write: insert raw XML at the cursor. The XML is written as-is — ensure it is well-formed and uses only elements valid at that location.


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `xml` | `string` | Yes | XML content to insert |

| `insertType` | `string` | No | How to insert the XML (insertInline, insertBefore, insertAfter) |





#### `insertXmlAtXpath`

Low-level primitive: insert raw XML at an exact xpath. The XML is written as-is — ensure it is well-formed and uses only elements valid at that location.


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `xml` | `string` | Yes | XML content to insert |

| `xpath` | `string` | Yes | XPath expression for insertion location |





#### `replaceTextAtCursor`

SELECTION-ANCHORED write: replaces the user's current selection with new plain text. Requires an active selection.


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `text` | `string` | Yes | New text to replace selection |





#### `replaceTextAtCursorParagraph`

CURSOR-ANCHORED write: replaces the entire paragraph the cursor is in with new plain text.


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `text` | `string` | Yes | New text to replace paragraph |





#### `replaceTextAtXpath`

Low-level primitive: replace plain text at an exact document xpath. Requires knowing the document structure.


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

Low-level primitive: replace raw XML at an exact xpath. The XML is written as-is — ensure it is well-formed; a malformed write can corrupt the document.


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

Tools for showing progress, results, and structured data to the user in the Flow Execution Monitor.



### Tools


#### `addFeedback`

Show a message in the execution monitor. Content is rendered as markdown. Use type "status"/"info" for progress notes and "success"/"warning"/"error" for outcomes.


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `type` | `string` | Yes | Type of feedback |

| `content` | `string` | Yes | Feedback message content |

| `metadata` | `object` | No | Optional metadata for the feedback |





#### `updateProgress`

Update the monitor progress bar (0-100). Use during long multi-step work so the user sees movement.


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

Display raw structured data as formatted JSON in the monitor. For human-readable results prefer addFeedback (markdown) or addTableFeedback.


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `data` | `object` | Yes | Data to display as JSON (object or array) |

| `title` | `string` | No | Optional title |

| `options` | `object` | No | Display options (e.g., maxDepth) |





#### `addKeyValueFeedback`

Display key-value pairs in the monitor — ideal for small fact sheets (e.g. author, date, status).


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `data` | `object` | Yes | Key-value pairs to display |

| `title` | `string` | No | Optional title |





#### `addMetricsFeedback`

Display numeric scores with labels in the monitor. Use for metrics and ratings; for arbitrary facts use addKeyValueFeedback.


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `metrics` | `object` | Yes | Metrics to display as key-value pairs (object: `{label: value}`) |

| `title` | `string` | No | Optional title |





#### `showNotification`

Show a system notification outside the monitor. Use sparingly for important outcomes — routine progress belongs in addFeedback.


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `type` | `string` | Yes | Notification type |

| `message` | `string` | Yes | Notification message |

| `title` | `string` | No | Optional notification title |





#### `highlightTextSegments`

Highlight text segments in the document by MATCHING THEIR CONTENT — no offsets needed. The preferred way to visually mark text; matching tolerance (case, whitespace) is configurable per segment.


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `segments` | `array` | Yes | Segments to highlight |

| `virtual` | `boolean` | No | Virtual tags (Swing only) |

| `tag` | `string` | No | Override tag to apply to all segments |

| `match` | `object` | No | Override match options to apply to all segments |

| `attributes` | `object` | No | Override attributes to apply to all segments |






## HighlightTools

Low-level text-tagging primitives operating on the document's normalized pure text. For marking text by content, prefer highlightTextSegments (FeedbackTools) or markSegments (StoryEditor).



### Tools


#### `getPureText`

Read the document as normalized plain text without markup. This is the exact text that pure-text tag offsets are computed against — read it before using addPureTextTags.


**Parameters:** None




#### `addPureTextTags`

OFFSET-LEVEL primitive: add tags at character offsets into the pure text. Offsets must be computed against getPureText exactly — prefer highlightTextSegments, which matches by text content instead of offsets.


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `tags` | `array` | Yes | Array of tags to add |

| `virtual` | `boolean` | No | Whether tags are virtual and will not be persisted(optional) |





#### `clearPureTextTags`

Remove all tags with the given names from the document pure text (e.g. to clear highlights added earlier).


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `tags` | `array` | Yes | Array of tags to be removed |






## StoryEditor

Task-level tools for reading, editing, and annotating the current story. The preferred tool surface for editorial agents — safe validated edits, named document fields, and visual marks.


### Configuration

Set via the `config:` block on the `builtinTools` class entry:

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `bodyXpath` | `string` | `"/doc/story"` | Content root read and written by searchAndReplaceText and the story-block tools |
| `appendXpath` | `string` | `"/doc/story/text"` | XPath of the element appendParagraph inserts new paragraphs into |
| `allowedElements` | `array` | `["p","h2","h3","ul","li","b","i","u"]` | XML elements allowed in searchAndReplaceText replacement content |
| `inlineElements` | `array` | `["b","i","u"]` | Inline formatting elements allowed in appendParagraph text and visible/editable in story blocks |
| `blockElements` | `array` | `[]` | Elements that form editable blocks; empty = automatic detection |
| `ignoreElements` | `array` | `[]` | Elements whose content is never shown or edited |
| `insertElements` | `array` | `[]` | Extra element names applyTextEdits may create as new blocks or retype to, beyond the types already used under the same parent |
| `reviewEdits` | `boolean` | `false` | Every write by applyTextEdits/applyProposedEdits first shows the Flow Monitor checklist; the user decides what is applied |
| `atomicEdits` | `boolean` | `true` | applyTextEdits applies all edits or none, so the agent can fix failures and resend the batch |
| `allowedOps` | `array` | `["replace","rewrite","comment","insert_before","insert_after","delete","split","merge","move_before","move_after","retype"]` | Edit operations applyTextEdits/proposeTextEdits accept; others fail with op_not_allowed |
| `fields` | `object` | — | Map of semantic field names to xpaths (e.g. headline: /doc/story/grouphead/headline/p). Generates the getField/setField tools with the field names as an enum; both tools are dropped when no fields are configured. |
| `markTag` | `string` | `"mark"` | Tag name used by markSegments/clearMarks |



### Tools


#### `readStory`

Read the current story as plain text. Use this before answering questions about the story and before any edit, so search strings can be copied exactly from the article.


**Parameters:** None




#### `readSelection`

Get the text the user has currently selected in the editor. Use when the user refers to "this", "the selected text", or "what I marked". Returns empty when nothing is selected.


**Parameters:** None




#### `readCurrentParagraph`

Get the text of the paragraph the user's cursor is in. ONLY useful when the user refers to "this paragraph" or "the current paragraph" — for any other location use readStory.


**Parameters:** None




#### `replaceCurrentParagraph`

Replace the full text of the paragraph the user's cursor is in. ONLY works on that one paragraph — for edits anywhere else in the story use searchAndReplaceText.


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `text` | `string` | Yes | New text for the paragraph |




**Returns:** JSON: \{ success: true \}



#### `searchAndReplaceText`

The preferred way to change article text — use this for removing, rewriting, or correcting any sentence or segment in the story. Use an empty replace string to delete the found text. Searches for an exact string and replaces its first occurrence. The search string must appear verbatim in the article — copy it exactly from the story text (or from the raw story XML when the search needs to include tags for structural changes). The replacement is validated before it is applied; if it would break the document, nothing is changed and an error with a hint is returned. Returns \{ found, success \} plus error/hint details on rejection.


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `search` | `string` | Yes | Exact text to find (copied verbatim from the article, may include XML tags) |

| `replace` | `string` | Yes | Replacement text (empty string deletes the found text) |




**Returns:** JSON: \{ found, success, error?, hint? \} — on rejection the document is unchanged



#### `appendParagraph`

Append the given text as a new paragraph at the end of the story. Inline formatting tags (by default \<b>, \<i>, \<u>) are allowed. Invalid markup is stripped automatically and the text is inserted as plain content (reported via note: 'invalid_markup_stripped').


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `text` | `string` | Yes | Paragraph text, optionally with allowed inline formatting tags |




**Returns:** JSON: \{ success, note?, error?, hint? \} — note 'invalid_markup_stripped' when markup was removed



#### `getField`

Read the text value of a named document field. Use this for the listed fields instead of xpath-based reads.


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `name` | `string` | Yes | Field name (see the listed available fields) |




**Returns:** The field text, or JSON \{ error: "unknown_field", hint \} for unconfigured fields



#### `setField`

Replace the text value of a named document field. Text-level write — use this for the listed fields instead of xpath-based or search-and-replace edits.


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `name` | `string` | Yes | Field name (see the listed available fields) |

| `text` | `string` | Yes | New text value for the field |




**Returns:** JSON: \{ success, field \} or \{ error: "unknown_field", hint \} for unconfigured fields



#### `markSegments`

Visually highlight text segments in the story (e.g. to show the user which sentences you propose to change). Does NOT modify the content. Matching is whitespace- and case-tolerant; pass the exact text of each segment as it appears in the story.


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `segments` | `array` | Yes | Segments to mark |




**Returns:** JSON result of the marking operation



#### `clearMarks`

Remove all highlight marks previously added with markSegments.


**Parameters:** None




#### `readStoryBlocks`

Read the story as numbered blocks "[b1 headline/p] …" (id, then the element role). Inline formatting tags shown in a block (e.g. \<b>, \<i>) are real; everything else is protected. Use before applyTextEdits/proposeTextEdits and copy "find" strings exactly from here.


**Parameters:** None




#### `applyTextEdits`

Edit the story safely in one batch, using ids from readStoryBlocks. Text: replace text/inline formatting inside a block, or rewrite a whole block (only the differences are applied). Structure: insert, delete, split, merge, move, or retype blocks. These move existing content, so links, captions, and embedded objects are preserved. Comment: flag text for the editor without changing it. With atomic batches (default) either all edits apply or none, and the result explains each failure with a hint; fix them and resend.


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `edits` | `array` | Yes | Edits to apply, in order |




**Returns:** JSON TextEditReport: \{ mode, applied, commented, unchanged, rejected, failed, results[], findings[] \}



#### `proposeTextEdits`

Suggest edits without changing the story: the findings are highlighted in the editor and returned as a numbered list (f1, f2, …). Present them to the user, then call applyProposedEdits with the ids the user accepts. Fix failed edits and propose them again.


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `edits` | `array` | Yes | Edits to propose, in order |




**Returns:** JSON: \{ proposalId, findings: [\{ id, label, block, reason \}], failed: [\{ index, block, error, hint \}] \}



#### `applyProposedEdits`

Apply the findings the user accepted from a proposeTextEdits proposal. Pass an empty array to discard the proposal and clear its highlights.


**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|

| `proposalId` | `string` | Yes | Id returned by proposeTextEdits |

| `findingIds` | `array` | No | Finding ids to apply (e.g. ["f1","f3"]); omit to apply all |




**Returns:** JSON TextEditReport: \{ mode, applied, commented, unchanged, rejected, failed, results[], findings[] \}




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

Tools for asking the user questions and decisions in the Flow Execution Monitor. Execution pauses until the user answers.



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






