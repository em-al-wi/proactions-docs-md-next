# Editor Steps


Steps in the Editor category.


## Available Steps


### [APPLY_TEXT_CUTS](./steps/APPLY_TEXT_CUTS.md)

Cuts a story to length from LLM cut candidates: every candidate is measured, alternatives and nested removals are detected, and the cuts needed for the target are preselected. In review mode the Flow Monitor shows a cut panel with a live meter and editor highlights, and writes the chosen cuts in one validated batch.




**Version:** 1.0.0



**Example:**
```yaml

 - step: GET_STORY_BLOCKS
   showLength: true
   outputs: [{ type: text, name: blocks }]
 - step: MONITOR_INPUT
   message: "How many characters must be cut?"
   required: true
   outputs: [{ type: text, name: cutByText }]
 - step: OPENAI_COMPLETION
   response_format: text_cuts
   behavior: |
     You are a newspaper editor proposing cuts. Never change facts, names, numbers or quotes.
     Prefer deleting low-value blocks and sentences over rewriting. Give every cut a priority
     and put alternative cuts of the same passage in one group.
   instruction: "{{ flowContext.blocks }}"
 - step: APPLY_TEXT_CUTS
   mode: review
   cutBy: "{{ flowContext.cutByText }}"
   charsPerLine: 34
   allowMore: true

```


---


### [APPLY_TEXT_EDITS](./steps/APPLY_TEXT_EDITS.md)

Applies LLM edits to the story through the story-block engine: text and inline formatting inside a block, whole-block rewrites, comments, and structural operations (insert, delete, split, merge, move, retype). Hidden markup — links, captions, embedded objects, editor placeholders — is preserved. Invalid edits change nothing and are reported with a hint.




**Version:** 1.0.0



**Example:**
```yaml

 - step: GET_STORY_BLOCKS
 - step: OPENAI_COMPLETION
   response_format: text_edits
   behavior: |
     You are a proofreader. Fix spelling and grammar only. Never change meaning or formatting.
     Use op "rewrite" for blocks with several fixes and op "replace" for single words.
   instruction: "{{ flowContext.text }}"
 - step: APPLY_TEXT_EDITS
   mode: review
   allowedOps: [replace, rewrite, comment]

```


---


### [CLEAR_SELECTION](./steps/CLEAR_SELECTION.md)

Clears the current text selection and optionally repositions the cursor to the start or end of the selection. Available in Swing editor context only.




**Version:** 1.0.7



**Example:**
```yaml

- step: CLEAR_SELECTION
  cursorPosition: "anchorEnd"

```


---


### [CLEAR_TAGS](./steps/CLEAR_TAGS.md)

Removes all occurrences of the specified tags from the document or report while preserving their text content. This is useful for cleaning up temporary markup or removing specific tag types.




**Version:** 1.2.0



**Example:**
```yaml

- step: CLEAR_TAGS
  tags: ["mark", "highlight"]

```


---


### [CLOSE_DOCUMENT](./steps/CLOSE_DOCUMENT.md)

Closes the current document without saving.




**Version:** 1.2.0



**Example:**
```yaml

- step: CLOSE_DOCUMENT

```


---


### [GET_PURE_TEXT](./steps/GET_PURE_TEXT.md)

Retrieves the full pure text content from the current document and sets it as output.




**Version:** 1.2.0



**Example:**
```yaml

 - step: GET_PURE_TEXT

```


---


### [GET_STORY_BLOCKS](./steps/GET_STORY_BLOCKS.md)

Reads the document as numbered, role-labelled text blocks for whole-document LLM use cases (proofreading, spell checking, style fixes). Inline formatting such as \<b> is shown, all other markup is hidden and protected. Feed the output to an LLM and apply its answer with APPLY_TEXT_EDITS.




**Version:** 1.0.0



**Example:**
```yaml

 - step: GET_STORY_BLOCKS

```


---


### [GET_TEXT_CONTENT](./steps/GET_TEXT_CONTENT.md)

Retrieves text content from the document or a specific location and sets it as output.


**Aliases:** CTX_GET_TEXT_CONTENT



**Version:** 1.0.1



**Example:**
```yaml

 - step: GET_TEXT_CONTENT
 
```


---


### [GET_XML_CONTENT](./steps/GET_XML_CONTENT.md)

Retrieves XML content from the document or a specific location and sets it as output.


**Aliases:** CTX_GET_XML_CONTENT



**Version:** 1.0.1



**Example:**
```yaml

 - step: GET_XML_CONTENT
 
```


---


### [GET_ZONES](./steps/GET_ZONES.md)

Retrieves zones and their linked documents from a DWP or Report.




**Version:** 1.0.0



**Example:**
```yaml

 - step: GET_ZONES
 
```


---


### [INSERT_LIST](./steps/INSERT_LIST.md)

Insert a list of items into the document. The items can be provided as a list input or be generated from a text input (converted to a list using ProcessToList).


**Aliases:** CTX_INSERT_LIST



**Version:** 1.0.0



**Example:**
```yaml

- step: SET
  text: "Item 1\\nItem 2\\nItem 3"

- step: INSERT_LIST
  containerElement: "ol"
  listItemElement: "li"

```


---


### [INSERT_OBJECT](./steps/INSERT_OBJECT.md)

Insert object references or templates into the current document using a DTX template.

Prime: Supports paths, UUIDs, and LOIDs (auto-formatted). Options insertionType, full, and templateType are ignored (warnings logged).




**Version:** 1.2.0



**Example:**
```yaml

- step: INSERT_OBJECT
  path: "/content/picture/chart.png"

```


---


### [INSERT_TEXT](./steps/INSERT_TEXT.md)

Insert text at the given location (cursor, xpath or report). If the step reads text from the flowContext, you can use a preceding SET step to provide the content.


**Aliases:** CTX_INSERT_TEXT



**Version:** 1.0.0



**Example:**
```yaml

- step: SET
  text: "This text will be inserted"
- step: INSERT_TEXT
  at: CURSOR

```


---


### [INSERT_XML](./steps/INSERT_XML.md)

Insert XML content at the given location. The content can be provided via the flowContext (using `in`) or via the default text input.


**Aliases:** CTX_INSERT_XML



**Version:** 1.0.0



**Example:**
```yaml

- step: SET
  text: "<p>Inserted paragraph</p>"

- step: INSERT_XML

```


---


### [REPLACE_TEXT](./steps/REPLACE_TEXT.md)

Replace existing text at the given location (cursor paragraph, cursor or xpath) with the provided content. Content can come from a flow variable (using `in`) or the default text input.


**Aliases:** CTX_REPLACE_TEXT



**Version:** 1.0.0



**Example:**
```yaml

- step: SET
  text: "Replacement text"

- step: REPLACE_TEXT
  at: CURSOR_PARAGRAPH

```


---


### [REPLACE_XML](./steps/REPLACE_XML.md)

Replace XML content at the given location. Content can be sourced from a flow variable (`in`) or the default text input.


**Aliases:** CTX_REPLACE_XML



**Version:** 1.0.0



**Example:**
```yaml

- step: SET
  text: "<section>Updated</section>"

- step: REPLACE_XML
  at: XPATH
  xpath: "/doc/story/grouphead/headline/p"

```


---


### [SAVE_DOCUMENT](./steps/SAVE_DOCUMENT.md)

Saves the current document. Optionally closes the document after saving.




**Version:** 1.2.0



**Example:**
```yaml

- step: SAVE_DOCUMENT

```


---


### [SELECTED_OBJECT](./steps/SELECTED_OBJECT.md)

Retrieves information about the currently selected content object and sets it as output.


**Aliases:** CTX_SELECTED_OBJECT



**Version:** 1.0.4



**Example:**
```yaml

 - step: SELECTED_OBJECT
 
```


---


### [SET_METADATA](./steps/SET_METADATA.md)

Set a field value in the object panel using a selector or XPath. Supports tagsinput and plain fields.


**Aliases:** CTX_SET_METADATA



**Version:** 1.0.0



**Example:**
```yaml

  - step: SET
    text: "The value"
  - step: SET_METADATA
   selector: "#metadata-title"
 
```


---


### [TAG_PURE_TEXT](./steps/TAG_PURE_TEXT.md)

Adds tags to pure text content in the document. Tags can highlight or annotate specific text ranges. In Swing, tags can be virtual (temporary) or persistent. In Prime, tags are always persistent.




**Version:** 1.2.0



**Example:**
```yaml

- step: TAG_PURE_TEXT
  tags:
    - tag: "mark"
      start: 100
      size: 10

```


---



## Related Documentation

- [Step Library Overview](./overview.md) - Browse all available steps
- [Getting Started](../getting-started/your-first-flow.md) - Learn how to create workflows
