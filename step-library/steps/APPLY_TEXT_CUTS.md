---
sidebar_position: 1
---

# APPLY_TEXT_CUTS

Cuts a story to length from LLM cut candidates: every candidate is measured, alternatives and nested removals are detected, and the cuts needed for the target are preselected. In review mode the Flow Monitor shows a cut panel with a live meter and editor highlights, and writes the chosen cuts in one validated batch.

## At a glance
- **Category** Editor
- **Version:** 1.0.0
- **Applications:** all
- **Scope:** all

## When to Use

Use **APPLY_TEXT_CUTS** after an LLM step answered with `response_format: text_cuts` for a story read with GET_STORY_BLOCKS (with `showLength: true`, so the model can size its cuts). Give the target as `cutBy` (characters to remove, e.g. the print overmatter) or `targetLength` (desired characters).

## Data Flow

Reads the cuts (or a proposal from an earlier round) from the object input. Writes the proposal in preview mode and the report otherwise to `object`; `previous` names a flowContext path holding an earlier proposal to merge.

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `mode` | review = let the user choose the cuts in the Flow Monitor cut panel. preview = report and highlight the preselection without writing (allowed on read-only documents). apply = write the preselection, or the ids in `selection`. | review |false| false |None|None|
| `cutBy` | Characters to remove; takes precedence over targetLength | None |false| true |None|None|
| `targetLength` | Desired story length in characters (current length − targetLength = cutBy) | None |false| true |None|None|
| `charsPerLine` | Characters per line for the line estimate in the meter; 0 = no estimate | 0 |false| false |None|None|
| `selection` | apply mode: cut ids to write (["c1","c3"], c1,c3 or a template), or auto for the preselection. Default: auto | auto |false| true |None|None|
| `allowMore` | Show an "Ask for more cuts" button in the review panel, so another round can add candidates | false |false| false |None|None|
| `previous` | flowContext path of an earlier TextCutProposal whose cuts are merged into this round and keep their ids | None |false| false |None|None|
| `mark` | Highlight the selected cuts in the editor (preview and review). false = no highlights: the cut panel alone shows the cuts and the story is unchanged until Apply. In Prime highlights are stored in the document | true |false| false |None|None|
| `markTag` | Tag name used for the cut highlights | mark |false| false |None|None|
| `forceWrite` | Bypass the read-only check for apply/review | None |false| false |None|None|
| `xpath` | XPath of the content root that is read and written (must match the GET_STORY_BLOCKS read) | /doc/story |false| false |None|None|
| `blockElements` | Elements that form editable blocks; empty = automatic detection. Must match the read |  |false| false |None|None|
| `ignoreElements` | Elements whose content is never shown or edited, e.g. byline. Must match the read |  |false| false |None|None|
| `inlineElements` | Formatting elements visible to and editable by the LLM. Must match the read | b,i,u |false| false |None|None|

### Step-Level Validation Rules

## Inputs
| Type | Description | Default | Required | Resolved |
|---|---|:---:|:---:|:---:|
| `object` | The `text_cuts` output of an LLM step, or a TextCutProposal from an earlier round. | None | true | false |

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `object` | preview: the TextCutProposal; otherwise the TextCutReport. | false |

## Examples

### Cut a story in the cut panel
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

### Report the cuts without writing
```yaml
 - step: GET_STORY_BLOCKS
   showLength: true
 - step: OPENAI_COMPLETION
   response_format: text_cuts
   instruction: "{{ flowContext.text }}"
 - step: APPLY_TEXT_CUTS
   mode: preview
   cutBy: 800
   outputs: [{ type: object, name: cuts }]
 - step: MONITOR_TABLE
   title: "Cut candidates"
   data: "{{ flowContext.cuts.candidates }}"
```

### Cut automatically to the target length
```yaml
 - step: APPLY_TEXT_CUTS
   mode: apply
   targetLength: 4500
   selection: auto
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
