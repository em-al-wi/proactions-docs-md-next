# Story Text Edits

This guide covers whole-story editing with an LLM: an AI step reads the story as numbered blocks, the model answers with a structured list of edits, and `APPLY_TEXT_EDITS` validates and applies them — leaving links, captions, embedded objects and placeholders untouched. Use it for proofreading, spelling and style fixes, structural changes (subheadings, splits, merges) and fact-check notes.

## Overview

Three pieces work together:

- **`GET_STORY_BLOCKS`** — reads the document as numbered, role-labelled text blocks.
- **`response_format: text_edits`** — makes the completion step answer with a strict JSON edit list instead of prose.
- **`APPLY_TEXT_EDITS`** — previews, applies, or reviews those edits in the Flow Monitor; invalid edits change nothing and are reported with a reason.

Agents connected through [StoryEditor tools](./builtin-ai-tools.md#storyeditor) get the same engine as `readStoryBlocks`, `applyTextEdits`, `proposeTextEdits` and `applyProposedEdits` — no flow authoring required.

## How It Works

1. **Read** — `GET_STORY_BLOCKS` renders the content root as one line per block (`[b1 headline/p] …`) and remembers each block's text as the *stale-block guard*.
2. **Ask** — a completion step with `response_format: text_edits` receives that text and answers with `{"edits": […]}`. The parsed answer lands in `flowContext.object`; the raw JSON stays in `flowContext.text`.
3. **Apply** — `APPLY_TEXT_EDITS` previews, applies, or reviews the edits. Every edit is resolved against the real XML, so text is inserted in the right nodes and hidden markup is preserved.

The minimal flow:

```yaml
- step: GET_STORY_BLOCKS
- step: HUB_COMPLETION
  response_format: text_edits
  behavior: |
    You are a proofreader. Fix spelling and grammar only. Never change meaning or formatting.
    Use op "rewrite" for blocks with several fixes and op "replace" for single words.
    Return an empty edits array when the story is already correct.
  instruction: "{{ flowContext.text }}"
- step: APPLY_TEXT_EDITS
  mode: review
  allowedOps: [replace, rewrite, comment]
```

`OPENAI_COMPLETION`, `HUB_COMPLETION` and `AZURE_OPENAI_COMPLETION` all accept `response_format: text_edits`. Review mode needs the Flow Monitor (`monitor: enabled: true` on the section or action).

## The Block View

For a typical story, `GET_STORY_BLOCKS` produces:

```text
[b1 headline/p] Probe reaches asteroid Bennu
[b2 summary/p] The contact lasted only a few seconds.
[b3 text/p] The probe Osiris made <b>breif</b> contact with the surface on Tuesday.
[b4 text/p] Scientists will know on Saturday wether the sample was collected.
[b5 text/p] NASA says the mission cost 800 million dollars.
```

What the model sees, and what it does not:

- **Ids** are `b1`, `b2`, … in document order. They are the only block references an edit may use, and they are deterministic for the same XML and the same options.
- **Roles** are `parentLocalName/elementLocalName` (for example `headline/p`, `text/p`), so the model can tell a headline from a body paragraph without knowing the DTD.
- **Blocks** are the nearest non-inline ancestor of text. Whitespace-only blocks are omitted entirely.
- **Formatting** in `inlineElements` (default `b`, `i`, `u`) is shown as literal tags when the element carries no attributes. Everything else — links, captions, embedded objects, placeholders, formatting with attributes — is hidden but kept: its text may be visible, its markup never is.
- **`ignoreElements`** subtrees are neither shown nor editable (byline, header, footer).
- **Escaping**: `&`, `<` and `>` inside text appear escaped in the block view.

Configure reading with these options, and use the same values on `APPLY_TEXT_EDITS`:

| Option | Default | Meaning |
| --- | --- | --- |
| `xpath` | `/doc/story` | Content root that is read and written |
| `blockElements` | `[]` (auto) | Elements that form editable blocks; empty means auto-detection by the nearest non-inline ancestor |
| `ignoreElements` | `[]` | Elements never shown or edited, e.g. `byline`, `web-image-group` |
| `inlineElements` | `[b, i, u]` | Formatting tags visible to and editable by the model |

A customer DTD that stores articles differently:

```yaml
- step: GET_STORY_BLOCKS
  xpath: /doc/artikel
  ignoreElements: [byline, footer, header]
  inlineElements: [b, i, u, em, strong]
```

**Repeat the same four options on `APPLY_TEXT_EDITS`.** The guard snapshot is keyed by `xpath` plus `blockElements`, `ignoreElements` and `inlineElements`; a step configured differently sees a different block set and reports `stale_block` for edits it cannot place.

## The `text_edits` Answer

With `response_format: text_edits` the step sends a strict JSON schema named `text_edits`:

- The root object is `{ "edits": [...] }`; an empty array means nothing to change.
- Every item carries all seven string fields — `op`, `block`, `target`, `as`, `find`, `replace`, `reason` — and unused ones are `""`.
- No system prompt is injected. The operation rules live in the schema descriptions, so **`behavior` must state the editorial rules** you want enforced.

The answer for the sample story:

```json
{
  "edits": [
    { "op": "replace", "block": "b3", "target": "", "as": "", "find": "<b>breif</b>", "replace": "<b>brief</b>", "reason": "spelling" },
    { "op": "rewrite", "block": "b4", "target": "", "as": "", "find": "", "replace": "Scientists will know on Saturday whether the sample was collected.", "reason": "spelling" },
    { "op": "comment", "block": "b5", "target": "", "as": "", "find": "800 million dollars", "replace": "", "reason": "Verify the figure against the NASA press release." }
  ]
}
```

### Operations

| Op | Fields used | Effect |
| --- | --- | --- |
| `replace` | `block`, `find`, `replace` | Changes text inside a block. `find` must be the exact, unique text as shown in the block view, including visible inline tags. `replace: ""` deletes the matched text. |
| `rewrite` | `block`, `replace` | Gives the full corrected block in `replace`; only the differences are applied. Produces one finding per changed passage. |
| `comment` | `block`, `find`, `reason` | Flags `find` with the note in `reason`. Never changes text. |
| `insert_before` / `insert_after` | `block`, `replace`, `as` | Inserts a new block next to the anchor, with the text in `replace`. `as` is the new block's element type; empty means the anchor's type. |
| `delete` | `block` | Removes the block. |
| `split` | `block`, `find` | The new block begins at `find`; everything before it stays in the original block. |
| `merge` | `block` | Appends the directly following block in the same container to this one. |
| `move_before` / `move_after` | `block`, `target` | Moves `block` next to `target`. |
| `retype` | `block`, `as` | Changes the block's element type to `as`. |

Prefer `split`, `merge` and `move_*` over retyping text: they move the existing nodes, so links, captions and embedded objects inside them survive. Types allowed for insert and retype are the ones already used under the same parent plus whatever `insertElements` lists.

Restrict what the model can do with `allowedOps` — anything else fails with `op_not_allowed` instead of being applied. For proofreading, `[replace, rewrite, comment]` is the safe set.

## Apply, Preview, Review

| Mode | Writes? | Allowed on a read-only document? | Output |
| --- | --- | --- | --- |
| `preview` | never | yes | `TextEditProposal` |
| `apply` (default) | once, after validation | no (unless `forceWrite: true`) | `TextEditReport` |
| `review` | once, with what the user ticks | no (unless `forceWrite: true`) | `TextEditReport` |

**Selection.** When the input is an edits payload, `selection` is ignored — everything the model proposed is applied. When the input is a proposal from an earlier `preview`, `selection` chooses the finding ids to apply: omit it for all, pass `[]` to discard everything (each finding reports `rejected`). `selection` accepts an array, a comma-separated string or a template such as `"{{ flowContext.picked }}"`. Comment findings are always kept: they never write anything.

**Review.** `mode: review` shows a checklist in the Flow Monitor — *"Review the proposed changes — uncheck any you do not want to apply."* with **Apply selected** and **Cancel**. Comments are not selectable, and Cancel applies nothing. `selection` pre-checks ids. Review needs a writable document and an enabled Flow Monitor; otherwise the step fails with `Review mode requires the Flow Monitor` or `APPLY_TEXT_EDITS: It is not possible to write content in read-only mode!`.

**Atomic batches.** `atomic: true` makes the batch all-or-nothing: if one non-comment edit fails, nothing is written and the other edits report `batch_failed`. Use it when a partially applied correction would be worse than none.

**Highlights.** A preview marks its findings in the editor using `markTag` (default `mark`); `mark: false` turns that off. In Swing the marks are virtual editor marks, not content. In Prime they are stored in the story as `<mark>` elements: run `CLEAR_TAGS` before the story is saved, or set `mark: false`. Remove them with `CLEAR_TAGS` and `tags: ["mark"]` (or your configured `markTag`). Applying a marked proposal clears the preview marks first and then re-marks the remaining comments.

## Outputs

`preview` writes a **`TextEditProposal`** to `object`:

| Field | Contents |
| --- | --- |
| `kind` | Always `"text_edit_proposal"` |
| `xpath` | Content root the proposal belongs to |
| `edits` | Normalized edits, with block ids remapped by the stale-block guard |
| `snapshot` | Block id → plain text at preview time (the guard) |
| `findings` | One entry per finding |
| `table` | Display rows for `MONITOR_TABLE` |
| `report` | The underlying report |

Each finding has `id` (`f1`, `f2`, … in payload order), `edit` (index into `edits`), an optional `hunk` (rewrite passage index), `op`, `block`, `role`, `before`, `after`, `reason`, `label`, `kind` (`"edit"` or `"comment"`) and `marked`.

`table` rows have exactly the columns `#`, `Block`, `Original`, `Suggestion`, `Reason`.

`apply` and `review` write a **`TextEditReport`**: `mode`, `proposed`, `applied`, `commented`, `unchanged`, `rejected`, `failed`, `results[]` and `findings[]`. A row in `results[]` is `{ index, op, block, find, status, error?, hint? }`, where `status` is one of `proposed`, `applied`, `commented`, `unchanged`, `rejected` or `failed`.

```yaml
- step: MONITOR_TABLE
  title: "Findings — uncheck anything you do not want"
  data: "{{ flowContext.object.table }}"     # after preview

- step: MONITOR_TABLE
  title: "Applied"
  data: "{{ flowContext.object.results }}"   # after apply / review
  maxRows: 30
```

## Examples

### Proofread with a checklist

Preview the findings, show them, then let the producer tick what to apply. The prompt is deliberately narrow: the schema carries the operation rules, not the editorial ones.

```yaml
- step: GET_STORY_BLOCKS
- step: HUB_COMPLETION
  response_format: text_edits
  behavior: |
    You are a meticulous newspaper proofreader working in the document's language.
    Fix only: spelling, grammar, punctuation, spacing, capitalisation, clear typos.
    Never rephrase, never translate, never change meaning, numbers, names, quotes or markup.
    Rewrite a whole block when it contains several fixes; use replace for single words.
    Put the rule you applied in "reason" (short: "spelling", "comma before a conjunction").
    Return an empty edits array when the story is already clean.
  instruction: "{{ flowContext.text }}"
- step: APPLY_TEXT_EDITS
  mode: preview
  mark: true
  allowedOps: [replace, rewrite, comment]
- step: IF
  condition: '{{ (flowContext.object.findings | length) > 0 }}'
  then:
    - step: MONITOR_TABLE
      title: "Findings — uncheck anything you do not want"
      data: "{{ flowContext.object.table }}"
    - step: APPLY_TEXT_EDITS
      mode: review
      atomic: true
      allowedOps: [replace, rewrite, comment]
    - step: MONITOR_TABLE
      title: "Applied"
      data: "{{ flowContext.object.results }}"
      maxRows: 30
  else:
    - step: SHOW_NOTIFICATION
      notificationType: success
      title: "Proofreading"
      message: "No spelling or grammar fixes to apply."
```

### Findings report on read-only stories

`preview` never writes, so reviewers can run it on documents they cannot change.

```yaml
- step: GET_STORY_BLOCKS
- step: HUB_COMPLETION
  response_format: text_edits
  behavior: |
    Report spelling, grammar and style issues, and flag anything a human must verify.
    Never change meaning, wording or markup.
  instruction: "{{ flowContext.text }}"
- step: APPLY_TEXT_EDITS
  mode: preview
  mark: true
  allowedOps: [replace, rewrite, comment]
- step: MONITOR_TABLE
  title: "Findings (highlights stay until CLEAR_TAGS)"
  data: "{{ flowContext.object.table }}"
```

### Fact-check notes only

Comments never change the document text, so this works on read-only stories too. In Prime, confirm **Clear highlights** before saving — **Keep highlights** leaves the `<mark>` elements in the story.

```yaml
- step: GET_STORY_BLOCKS
- step: HUB_COMPLETION
  response_format: text_edits
  behavior: |
    Flag text a human must verify: questionable facts, unattributed quotes, risky or misleading
    wording, names and numbers worth a second look.
    Use op "comment" only, one note per issue, and put the check to perform in "reason".
    Never suggest rewrites.
  instruction: "{{ flowContext.text }}"
- step: APPLY_TEXT_EDITS
  mode: preview
  mark: true
  allowedOps: [comment]
- step: MONITOR_TABLE
  title: "Notes to verify (reason = what to check)"
  data: "{{ flowContext.object.table }}"
- step: MONITOR_CONFIRM
  message: "Have you checked these notes?"
  confirm: "Clear highlights"
  cancel: "Keep highlights"
- step: IF
  condition: '{{ flowContext.confirmed }}'
  then:
    - step: CLEAR_TAGS
      tags: ["mark"]
```

`MONITOR_CONFIRM` writes the boolean `flowContext.confirmed`.

### Apply all fixes in one batch

Skip the checklist when the corrections are mechanical — but keep the batch atomic so a half-applied set never reaches the document.

```yaml
- step: GET_STORY_BLOCKS
- step: HUB_COMPLETION
  response_format: text_edits
  behavior: |
    Fix spelling and grammar only. Never change meaning, numbers, names or markup.
  instruction: "{{ flowContext.text }}"
- step: APPLY_TEXT_EDITS
  mode: apply
  atomic: true
  allowedOps: [replace, rewrite]
- step: MONITOR_TABLE
  title: "Applied"
  data: "{{ flowContext.object.results }}"
  maxRows: 30
```

When one edit fails, `atomic` marks every other edit in the batch `batch_failed`, so the report shows why the batch was not written.

### Let a script pick the findings

A preview proposal can be filtered in code — for example, apply only the single-word replacements and leave rewrites for a human.

```yaml
- step: GET_STORY_BLOCKS
- step: HUB_COMPLETION
  response_format: text_edits
  behavior: |
    Fix spelling and grammar. Use replace for single words and rewrite for blocks with several fixes.
  instruction: "{{ flowContext.text }}"
- step: APPLY_TEXT_EDITS
  mode: preview
  allowedOps: [replace, rewrite]
- step: SCRIPTING
  script: |
    const findings = (flowContext.object && flowContext.object.findings) || [];
    flowContext.picked = findings.filter((f) => f.op === 'replace').map((f) => f.id);
    return flowContext;
- step: APPLY_TEXT_EDITS
  mode: apply
  selection: "{{ flowContext.picked }}"
  allowedOps: [replace, rewrite]
- step: CLEAR_TAGS
  tags: ["mark"]
```

Passing `selection: []` discards the whole proposal and nothing is written.

### Add subheadings (structural edits)

Structural operations move existing nodes instead of retyping them, so nothing inside a paragraph is lost.

```yaml
- step: GET_STORY_BLOCKS
- step: HUB_COMPLETION
  response_format: text_edits
  behavior: |
    Insert a short subheading before each block that starts a new topic:
    op "insert_before", "as": "h2", the subheading in "replace".
    When a new topic starts inside a block, use op "split" with "find" = the text where the
    new block begins. Never change, delete or move existing text.
  instruction: "{{ flowContext.text }}"
- step: APPLY_TEXT_EDITS
  mode: review
  allowedOps: [insert_before, split]
  insertElements: [h2]
```

List in `insertElements` only elements your DTD allows under that parent; anything else fails with `invalid_element_type`.

### Agent with StoryEditor tools

The same engine is available to agents as tools. With `reviewEdits: true`, every write goes through the Flow Monitor checklist first:

```yaml
- step: HUB_COMPLETION
  maxToolIterations: 12
  behavior: You are an editorial assistant.
  instruction: |
    1. Call readStoryBlocks.
    2. Call applyTextEdits with spelling and grammar fixes only — the user confirms the
       changes in the Flow Monitor checklist. Report the result.
  builtinTools:
    - class: StoryEditor
      only: [readStoryBlocks, applyTextEdits, clearMarks]
      config:
        allowedOps: [replace, rewrite, comment]
        reviewEdits: true
```

For an interactive agent that proposes first and applies only what the user accepts:

```yaml
- step: HUB_COMPLETION
  maxToolIterations: 12
  toolChoice: required
  behavior: 'You are an editorial assistant. The user cannot type: every interaction uses tools.'
  instruction: |
    1. Call readStoryBlocks.
    2. Call proposeTextEdits with spelling and grammar fixes only (never restructure).
    3. Present the numbered findings and ask which ones to apply using askMultipleChoice.
    4. Call applyProposedEdits with the accepted ids.
    5. Call clearMarks when you are done.
  builtinTools:
    - class: StoryEditor
      only: [readStoryBlocks, proposeTextEdits, applyProposedEdits, clearMarks]
      config:
        allowedOps: [replace, rewrite, comment]
        reviewEdits: false
    - class: UserInteractionTools
      only: [askMultipleChoice]
```

See [Built-in AI Tools — StoryEditor](./builtin-ai-tools.md#storyeditor) for the full tool list and configuration. For agent tools, `atomicEdits` defaults to `true`, so a failed edit means the agent fixes it and resends the batch.

## Complete Section Example

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/em-al-wi/proactions-schema/main/schema/partial-section.schema.json
section: Proofreading
context: story
monitor:
  enabled: true
actions:
  - title: "Proofread story"
    flow:
      - step: GET_STORY_BLOCKS
        xpath: /doc/story
        ignoreElements: [byline, footer, header]
        inlineElements: [b, i, u, em, strong]
      - step: HUB_COMPLETION
        response_format: text_edits
        behavior: |
          You are a meticulous newspaper proofreader working in the document's language.
          Fix only: spelling, grammar, punctuation, spacing, capitalisation, clear typos.
          Never rephrase, never translate, never change meaning, numbers, names, quotes or markup.
          Rewrite a whole block when it contains several fixes; use replace for single words.
          Put the rule you applied in "reason" (short: "spelling", "comma before a conjunction").
          Return an empty edits array when the story is already clean.
        instruction: "{{ flowContext.text }}"
      - step: APPLY_TEXT_EDITS
        mode: preview
        mark: true
        xpath: /doc/story
        ignoreElements: [byline, footer, header]
        inlineElements: [b, i, u, em, strong]
        allowedOps: [replace, rewrite, comment]
      - step: IF
        condition: '{{ (flowContext.object.findings | length) > 0 }}'
        then:
          - step: MONITOR_TABLE
            title: "Findings — uncheck anything you do not want"
            data: "{{ flowContext.object.table }}"
          - step: APPLY_TEXT_EDITS
            mode: review
            atomic: true
            allowedOps: [replace, rewrite, comment]
          - step: MONITOR_TABLE
            title: "Applied"
            data: "{{ flowContext.object.results }}"
            maxRows: 30
        else:
          - step: SHOW_NOTIFICATION
            notificationType: success
            title: "Proofreading"
            message: "No spelling or grammar fixes to apply."

  - title: "Check facts & wording"
    flow:
      - step: GET_STORY_BLOCKS
        xpath: /doc/story
        ignoreElements: [byline, footer, header]
        inlineElements: [b, i, u, em, strong]
      - step: HUB_COMPLETION
        response_format: text_edits
        behavior: |
          Flag text a human must verify: questionable facts, unattributed quotes, risky or
          misleading wording, names and numbers worth a second look.
          Use op "comment" only, one note per issue, and put the check to perform in "reason".
          Never suggest rewrites.
        instruction: "{{ flowContext.text }}"
      - step: APPLY_TEXT_EDITS
        mode: preview
        mark: true
        xpath: /doc/story
        ignoreElements: [byline, footer, header]
        inlineElements: [b, i, u, em, strong]
        allowedOps: [comment]
      - step: MONITOR_TABLE
        title: "Notes to verify (reason = what to check)"
        data: "{{ flowContext.object.table }}"
      - step: MONITOR_CONFIRM
        message: "Have you checked these notes?"
        confirm: "Clear highlights"
        cancel: "Keep highlights"
      - step: IF
        condition: '{{ flowContext.confirmed }}'
        then:
          - step: CLEAR_TAGS
            tags: ["mark"]
```

## Error Codes

A rejected edit never changes the document. Each failed row in `results[]` carries an `error` and a `hint` written for the model, so an agent can resend a corrected batch.

| Code | Meaning | What to do |
| --- | --- | --- |
| `missing_params` | A field the op needs is empty. | Require `block`, `find`, `replace` or `target` in `behavior`. |
| `invalid_op` | The op name is not one of the eleven operations. | Spell the op exactly, or widen `allowedOps`. |
| `op_not_allowed` | The op is valid but excluded by `allowedOps`. | Widen `allowedOps`, or make the prompt stop asking for it. |
| `unknown_block` | The `block` id is not in the read. | Re-run `GET_STORY_BLOCKS` and use the ids it produced. |
| `stale_block` | The block's text changed since the read (or the two steps use different options). | Re-run `GET_STORY_BLOCKS`; use identical options on both steps. |
| `not_found` | `find` does not occur in the block. | `find` must be copied verbatim, including visible inline tags. |
| `ambiguous_match` | `find` occurs more than once in the block. | Extend `find` with surrounding words so it is unique. |
| `overlapping_edit` | Two edits claim the same text. | Apply one, or merge them into a single `rewrite`. |
| `search_matches_markup` | `find` cuts an escaped character, an unbalanced tag pair, or crosses formatting boundaries. | Keep `find` inside one text run; include complete tag pairs. |
| `protected_markup` | The edit would remove or rewrite hidden content (links, embedded objects, formatting with attributes). | Narrow `find`, or use `split`/`move` so the markup moves intact. |
| `invalid_element_type` | The requested `as` type is not used under that parent. | Add the type to `insertElements`, or pick an existing type. |
| `invalid_position` | The block has no parent element, so it cannot be moved or inserted next to. | Target a block inside a container. |
| `structure_required` | The block is the only element of its type in its parent and must stay. | Change its text with `replace`/`rewrite` instead of deleting or splitting it. |
| `batch_failed` | Another edit in the same `atomic` batch failed, so this one was not applied. | Fix the failing edit and resend the whole batch. |
| `escaped_markup`, `replacement_not_well_formed`, `disallowed_elements`, `disallowed_attributes`, `disallowed_node_type` | The replacement content is not valid for the document: re-escaped tags, broken XML, or elements/attributes that are not allowed. | Return plain text plus the inline tags shown in the block view. |
| `not_markable` | The finding cannot be highlighted in the editor (nothing to mark at that position). | Harmless: the edit is still applied; only the highlight is skipped. |
| `unknown_finding` | `selection` names a finding id that is not in the proposal. | Use the ids from `object.findings` (`f1`, `f2`, …). |

Step-level failures stop the flow instead of producing rows: `GET_STORY_BLOCKS: No XML found at xpath "<xpath>"`, `edits must be an array or an object with an "edits" array`, the read-only error, and `Review mode requires the Flow Monitor`.

## Troubleshooting

- **`Review mode requires the Flow Monitor`** — set `monitor: enabled: true` on the section or action, or use `mode: preview`.
- **`It is not possible to write content in read-only mode!`** — use `mode: preview`, or set `forceWrite: true` if the flow is trusted to write anyway.
- **Many `stale_block` errors** — the story changed after `GET_STORY_BLOCKS`, or the two steps use different `xpath`, `blockElements`, `ignoreElements` or `inlineElements`. Re-run the read with matching options.
- **`not_found` for text you can see** — the model re-worded or re-escaped `find`. Tell it to copy the text verbatim, including visible inline tags such as `<b>`.
- **`No XML found at xpath`** — your DTD stores content elsewhere; set `xpath` on both steps.
- **Highlights stay in the editor** — marks are virtual in Swing and stored as `<mark>` elements in Prime; clear them with `CLEAR_TAGS` and `tags: ["mark"]` (or your `markTag`). Applying a marked proposal clears them automatically.
- **HTTP 400 about `temperature`** — reasoning models reject it; omit `options` on the completion step.
- **HTTP 400 about `response_format` or `json_schema`** — `text_edits` uses OpenAI structured outputs; the model, Azure deployment or Hub target must support `json_schema` with `strict: true`.
- **No streaming** — streaming is disabled for `json_schema` response formats, so the text output arrives in one piece.

## See Also

- [`GET_STORY_BLOCKS`](../../step-library/steps/GET_STORY_BLOCKS.md)
- [`APPLY_TEXT_EDITS`](../../step-library/steps/APPLY_TEXT_EDITS.md)
- [Cut to Length](./text-cuts.md) — shorten a story to a target length with ranked LLM cuts and a selection panel
- [Built-in AI Tools — StoryEditor](./builtin-ai-tools.md#storyeditor)
- [Flow Monitor](./flow-monitor.md)
- [AI Integration](./ai-integration.md)
