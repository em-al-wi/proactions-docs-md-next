---
sidebar_position: 1
---

# GET_STORY_BLOCKS

Reads the document as numbered, role-labelled text blocks for whole-document LLM use cases (proofreading, spell checking, style fixes). Inline formatting such as \<b> is shown, all other markup is hidden and protected. Feed the output to an LLM and apply its answer with APPLY_TEXT_EDITS.

## At a glance
- **Category** Editor
- **Version:** 1.0.0
- **Applications:** all
- **Scope:** all

## When to Use

Use **GET_STORY_BLOCKS** before any edit that an LLM should make on a whole story: the numbered blocks (b1, b2, …) are the ids APPLY_TEXT_EDITS expects, and the read is what the stale-block guard compares against.

## Data Flow

Reads no flow context values. Writes the rendered blocks to `text` (or its configured text output name).

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `xpath` | XPath of the content root that is read and written (e.g. /doc/story or /doc/artikel) | /doc/story |false| false |None|None|
| `blockElements` | Elements that form editable blocks. Empty (default) = automatic detection: the nearest non-inline ancestor of text, which works without knowing the customer DTD. |  |false| false |None|None|
| `ignoreElements` | Elements whose content is never shown or edited, e.g. byline, web-image-group |  |false| false |None|None|
| `inlineElements` | Formatting elements visible to and editable by the LLM | b,i,u |false| false |None|None|
| `insertElements` | Extra element names an insert or retype edit may use, in addition to the types already used under the same parent, e.g. h2 |  |false| false |None|None|
| `allowedOps` | Edit operations this step accepts; others fail with op_not_allowed | replace,rewrite,comment,insert_before,insert_after,delete,split,merge,move_before,move_after,retype |false| false |None|None|
| `showLength` | Append each block's visible character count to its line ("[b3 text/p · 412]") so an LLM can size cuts | false |false| false |None|None|

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `text` | The story as numbered blocks: one line per block, "[b1 headline/p] text". | false |
| `json` | Story length \{ chars, blocks: [\{ id, role, chars \}] \}; written only when an output of type json is configured | true |

## Examples

### Read the story as blocks for an LLM
```yaml
 - step: GET_STORY_BLOCKS
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
