---
sidebar_position: 1
---

# APPLY_TEXT_EDITS

Applies LLM edits to the story through the story-block engine: text and inline formatting inside a block, whole-block rewrites, comments, and structural operations (insert, delete, split, merge, move, retype). Hidden markup — links, captions, embedded objects, editor placeholders — is preserved. Invalid edits change nothing and are reported with a hint.

## At a glance
- **Category** Editor
- **Version:** 1.0.0
- **Applications:** all
- **Scope:** all

## When to Use

Use **APPLY_TEXT_EDITS** after an LLM step that answered with `response_format: text_edits` for a story read with GET_STORY_BLOCKS. Use mode `preview` for a findings report without writing, and mode `review` to let the user pick the changes in the Flow Monitor. Set `mark: false` to keep the story untouched until Apply (no highlights).

## Data Flow

Reads the edits (or a proposal from an earlier preview) from the object input. Writes the proposal (preview) or the report (apply/review) to `object`.

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `mode` | apply = write the validated edits. preview = report and highlight the findings without writing (allowed on read-only documents). review = let the user check the findings in the Flow Monitor. | apply |false| false |None|None|
| `mark` | Create editor highlights: the findings in preview and review, and comment notes. false = the story text is never tagged — review shows each finding with its surrounding text in the Flow Monitor, notes appear only in the monitor and the report (status rejected, error marks_disabled), and nothing changes until Apply. In Prime highlights are stored in the document until CLEAR_TAGS removes them | true |false| false |None|None|
| `selection` | Finding ids to apply when the input is a proposal; in review mode, the pre-checked ids. Accepts an array (["f1","f3"]), a comma-separated string (f1,f3) or a template (\{\{ flowContext.picked \}\}). Default: all | None |false| true |None|None|
| `xpath` | XPath of the content root that is read and written (e.g. /doc/story or /doc/artikel) | /doc/story |false| false |None|None|
| `blockElements` | Elements that form editable blocks. Empty (default) = automatic detection: the nearest non-inline ancestor of text, which works without knowing the customer DTD. |  |false| false |None|None|
| `ignoreElements` | Elements whose content is never shown or edited, e.g. byline, web-image-group |  |false| false |None|None|
| `inlineElements` | Formatting elements visible to and editable by the LLM | b,i,u |false| false |None|None|
| `insertElements` | Extra element names an insert or retype edit may use, in addition to the types already used under the same parent, e.g. h2 |  |false| false |None|None|
| `allowedOps` | Edit operations this step accepts; others fail with op_not_allowed. E.g. [replace, rewrite, comment] for proofreading. | replace,rewrite,comment,insert_before,insert_after,delete,split,merge,move_before,move_after,retype |false| false |None|None|
| `markTag` | Tag name used for the preview/review highlights | mark |false| false |None|None|
| `forceWrite` | Bypass the read-only check for apply/review | None |false| false |None|None|
| `atomic` | Apply all edits or none: if any edit fails, nothing is written and the report lists why | false |false| false |None|None|

## Inputs
| Type | Description | Default | Required | Resolved |
|---|---|:---:|:---:|:---:|
| `object` | The `text_edits` output of an LLM step, or a proposal from an earlier preview. | None | true | false |

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `object` | In preview mode the TextEditProposal; otherwise the TextEditReport. | false |

## Examples

### Spell-check with the built-in checklist
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

### Review without touching the story
```yaml
 - step: GET_STORY_BLOCKS
 - step: OPENAI_COMPLETION
   response_format: text_edits
   behavior: "You are a proofreader. Fix spelling and grammar only."
   instruction: "{{ flowContext.text }}"
 - step: APPLY_TEXT_EDITS
   mode: review
   mark: false
   allowedOps: [replace, rewrite, comment]
```

### Findings report without writing
```yaml
 - step: GET_STORY_BLOCKS
 - step: OPENAI_COMPLETION
   response_format: text_edits
   behavior: "You are a proofreader. Report spelling and grammar errors."
   instruction: "{{ flowContext.text }}"
 - step: APPLY_TEXT_EDITS
   mode: preview
   allowedOps: [replace, rewrite, comment]
 - step: MONITOR_TABLE
   title: "Spelling findings"
   data: "{{ flowContext.object.table }}"
```

### Apply a custom selection
```yaml
 - step: APPLY_TEXT_EDITS
   mode: apply
   selection: "{{ flowContext.selectedFindings }}"
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
