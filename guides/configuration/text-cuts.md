# Cut to Length

This guide covers shortening a story by a known amount — typically the overmatter of a print slot — with an LLM that proposes cuts and an editor who chooses them. `GET_STORY_BLOCKS` reads the story with a size per block, the model answers with ranked cut candidates, and `APPLY_TEXT_CUTS` measures every candidate, preselects enough of them to reach the target, and lets the editor pick the cuts in a Flow Monitor panel with a live meter and editor highlights.

## Overview

LLMs are good at deciding *what* can go and bad at hitting a length target. The work is split accordingly:

- **The model** only proposes cuts: delete a paragraph, drop or condense a passage, and alternatives for the same passage. It ranks them by priority.
- **ProActions** measures what each cut really saves, works out which cuts cannot be combined, and preselects cuts until the target is reached.
- **The editor** checks, unchecks or swaps cuts in the cut panel and applies the selection in one validated write.

Four pieces work together:

- **`GET_STORY_BLOCKS`** with `showLength: true` — reads the story as numbered blocks and appends each block's character count.
- **`response_format: text_cuts`** — makes the completion step answer with a strict JSON list of cut candidates.
- **`APPLY_TEXT_CUTS`** — measures, preselects, and then previews, applies, or opens the cut panel.
- **The cut panel** — a dedicated Flow Monitor view with a meter, the cuts grouped by paragraph, and live highlights in the editor.

The cut engine is built on the same block engine as [Story Text Edits](./story-text-edits.md): links, embedded objects, placeholders and strikeouts are protected, and a cut that would damage them is never offered.

:::tip Ready-to-use configuration
A complete, commented section — input in characters or lines, up to three rounds of "ask for more cuts", and an outcome message — is shown in [Complete Section Example](#complete-section-example). Copy it, adapt the values marked `◆ ADAPT`, and include it in `SECTIONS`.
:::

## How It Works

1. **Read** — `GET_STORY_BLOCKS` with `showLength: true` renders one line per block with its size (`[b3 text/p · 412] …`) and remembers each block's text as the stale-block guard.
2. **Ask** — a completion step with `response_format: text_cuts` receives the blocks and answers with `{"cuts": […]}`. The parsed answer lands in `flowContext.object`.
3. **Measure** — `APPLY_TEXT_CUTS` resolves every cut alone against a copy of the story and measures the characters it removes. Cuts that cannot be resolved, or that do not shorten the story, are reported in `failed[]` and never offered.
4. **Preselect** — candidates that cannot be combined are linked as conflicts, and the cheapest set of cuts that reaches the target is checked.
5. **Choose and apply** — in `review` mode the editor adjusts the selection in the cut panel; the chosen cuts are written in one atomic batch, and the story length is measured again afterwards.

## Reading With Sizes

`showLength: true` appends each block's visible character count to its line, so the model can size its cuts instead of guessing:

```text
[b1 text/p · 412] Major League Soccer's 2026 season has begun, with matches again streaming on <i>Apple TV</i>. …
[b2 text/p · 618] The agreement also extends to more than 300,000 bars, restaurants and hotels. …
[b3 text/p · 509] Apple TV, launched in 2019, has expanded its offerings beyond original films and series. …
```

Block ids are unchanged (`b1`, `b2`, …), so edits and cuts parse the same way with or without sizes.

Add an output of type `json` to also get the measured length as data:

```yaml
- step: GET_STORY_BLOCKS
  xpath: /doc/story/text
  showLength: true
  outputs:
    - type: text
      name: blocks
    - type: json
      name: storyLength # { chars: 1539, blocks: [{ id: "b1", role: "text/p", chars: 412 }, …] }
```

The `json` output is written only when it is configured, so existing flows are unaffected.

**Choose `xpath` to match the print frame.** The meter counts the characters of the blocks under `xpath`. If the overmatter refers to the body text, read only the body (Méthode story DTD: `/doc/story/text`); reading `/doc/story` would also count — and offer to cut — headline, summary and captions. Use the same `xpath`, `blockElements`, `ignoreElements` and `inlineElements` on `GET_STORY_BLOCKS` and `APPLY_TEXT_CUTS`, exactly as for [story text edits](./story-text-edits.md#the-block-view).

### How characters are counted

A block's size is its visible text with every whitespace run counted as one space and the ends trimmed. Inline tags, hidden markup, and strikeout text do not count. The story length is the sum over the blocks. It can therefore differ slightly from the editor's own character counter, which may count line breaks or hidden text differently — the meter and the report always use this one definition, so the numbers are comparable with each other.

## The `text_cuts` Answer

With `response_format: text_cuts` the completion step (`OPENAI_COMPLETION`, `HUB_COMPLETION`, `AZURE_OPENAI_COMPLETION`) sends a strict JSON schema named `text_cuts`:

- The root object is `{ "cuts": [...] }`; an empty array means nothing can be cut.
- Every item carries all seven fields — `op`, `block`, `find`, `replace`, `priority`, `group`, `reason` — and unused string fields are `""`.
- No system prompt is injected. The field rules live in the schema descriptions, so **`behavior` must state the editorial rules** (what may be cut, what must stay).

| Field | Meaning |
| --- | --- |
| `op` | `delete`, `replace` or `rewrite` (see below). |
| `block` | Block id, e.g. `b3`. |
| `find` | `replace` only: exact text copied from the block, unique within it, including inline tags. |
| `replace` | `replace`: the shorter text (`""` removes the passage). `rewrite`: the complete shortened block. |
| `priority` | `1` = least important, cut first; `2` = normal; `3` = cut only if needed. Any other value counts as `2`. |
| `group` | The same non-empty label on cuts that are alternatives for the same passage; at most one of them is applied. `""` when the cut has no alternative. |
| `reason` | Short explanation shown to the editor. |

| Op | Effect |
| --- | --- |
| `delete` | Removes the whole block. |
| `replace` | Replaces `find` with the shorter `replace`; an empty `replace` removes the passage. |
| `rewrite` | Replaces the block with the condensed version in `replace`; only the differences are applied. |

Every cut must make sense on its own: the editor may apply any combination of cuts that are not alternatives of each other.

An answer for the sample story above:

```json
{
  "cuts": [
    { "op": "replace", "block": "b2", "find": " and player-curated city guides", "replace": "", "priority": 1, "group": "guides", "reason": "secondary feature" },
    { "op": "rewrite", "block": "b2", "find": "", "replace": "The agreement also extends to bars and hotels. Apple Maps provides Matchday Guides.", "priority": 2, "group": "guides", "reason": "condense the paragraph" },
    { "op": "delete", "block": "b3", "find": "", "replace": "", "priority": 2, "group": "", "reason": "background on the service" }
  ]
}
```

The first two cuts are alternatives (group `guides`); the third is independent.

:::warning Pass the blocks with `| safe`
When the blocks are part of a longer template — for example an `instruction:` block with more text — write `{{ flowContext.blocks | safe }}`. Without `| safe`, the template engine escapes `<b>` to `&lt;b&gt;`, the model copies the escaped text into `find`, and the cuts fail with `not_found`. A template that consists only of `"{{ flowContext.blocks }}"` is passed through unchanged.
:::

## Setting the Target

`APPLY_TEXT_CUTS` takes the target in one of two units:

| Option | Meaning |
| --- | --- |
| `cutBy` | Characters to remove, e.g. the overmatter. |
| `targetLength` | The desired story length; `cutBy` becomes *current length − `targetLength`*. |

Set exactly one of them (they conflict in the schema). Both accept a number or a template such as `"{{ flowContext.cutBy }}"`; a value that does not resolve to a number fails the step with `APPLY_TEXT_CUTS: cutBy must be a number` (or `targetLength`).

When the input is a **proposal** from an earlier run (see [Outputs](#outputs)), both options are optional: without them the proposal's own target is used. A new target on a proposal re-runs the preselection only for `mode: apply` with an automatic `selection`; in review the editor sees the new target in the meter and adjusts.

If the story already fits (`cutBy` ≤ 0), the step reports `action: fits` and does nothing else.

**Lines instead of characters.** `charsPerLine` (default `0` = off) adds an estimated line count to the meter and the report. Use the average characters per line of your print column; the estimate is deliberately simple (each block rounds up to whole lines, headings are not weighted). To let editors enter the overmatter in lines, convert it in a `SCRIPTING` step — the [complete example](#complete-section-example) accepts both `350` and `12 lines`.

## How Cuts Are Measured and Selected

**Measurement.** Each cut is resolved and applied *alone* on a copy of the story, and its saving is the length difference. Because cuts are measured individually, the meter is exact for any combination of compatible cuts. A cut whose saving is not positive is reported as `no_saving`.

**Conflicts.** Two cuts cannot be applied together when:

- they share a non-empty `group` (alternatives), or
- the block engine reports that they overlap — the same text, a `rewrite` and another edit of the same block, or a `delete` of a block another cut edits.

Conflicts are symmetric. In the panel, checking a cut unchecks its conflicting cuts, so swapping an alternative is one click.

**Structure.** A `delete` may not remove the last element of its type under its parent (the story must keep at least one paragraph of each kind). The panel disables such a delete with the tooltip *Keep at least one block of this type*, and the preselection skips it.

**Preselection.** Starting from the lowest priority level that still has addable cuts, the smallest cut that covers the remaining amount is chosen; if none is large enough, the largest one is taken and the search continues. Ties go to the earlier cut. The result is deterministic: the same proposal always preselects the same cuts. Because priority 1 is used up first, the preselection may overshoot the target when the least important cut is large — the editor can swap it in the panel.

**Limits.** At most 60 cuts are accepted per round (the rest fail with `too_many_candidates`), and a cut identical to an earlier one fails with `duplicate`. Only `delete`, `replace` and `rewrite` are accepted; any other op fails with `op_not_allowed`.

## Review, Preview, Apply

| Mode | Writes? | Allowed on a read-only document? | Output |
| --- | --- | --- | --- |
| `review` (default) | once, with what the editor applies | no (unless `forceWrite: true`) | `TextCutReport` |
| `preview` | never | yes | `TextCutProposal` |
| `apply` | once, the preselection or `selection` | no (unless `forceWrite: true`) | `TextCutReport` |

Every write is atomic: either all selected cuts are applied or none. A selection that contains two alternatives, or deletes every block of one type, is refused before anything is written.

**`apply`.** Writes the preselection, or the ids in `selection` (an array such as `["c1","c3"]`, a comma-separated string, or a template). `selection: auto` (the default) and `all` both mean the preselection.

**`preview`.** Returns the proposal and, with `mark: true` (default), highlights the preselected cuts. In Swing the highlights are virtual; in Prime they are stored as `<mark>` elements until `CLEAR_TAGS` removes them.

### The cut panel

`mode: review` opens the cut panel in the Flow Monitor:

- **Header** — *Cut 300 characters: 1962 → 1662. Uncheck what must stay, check alternatives to swap.*
- **Meter** — *Cut 238 of 300 characters*, the new story length (*Story: 1962 → 1724 characters*), the estimated lines when `charsPerLine` is set, and either *Target reached* or *62 characters still missing*. The bar turns from the warning to the success colour when the target is reached.
- **Toolbar** — *Auto-select* restores the preselection; *Clear* unchecks everything.
- **Cuts** — grouped by paragraph (`b2 text/p · 618`), each with a checkbox, its priority (`P1`), its saving (`−126`), the removed text struck through and the replacement, the reason, and *Alternative to c2* where applicable.
- **Actions** — *Apply selected cuts*, *Ask for more cuts* (only with `allowMore: true`), and *Cancel*.

With `mark: true` (default) the checked cuts are highlighted in the editor and the highlights follow every checkbox change. They are removed when the panel closes, whatever the outcome. The meter, toolbar and row labels follow the ProActions language (English, German, French, Italian, Spanish); the header and the action buttons are in English.

The panel needs an enabled Flow Monitor (`monitor: enabled: true` on the section or action) and a writable document.

### More rounds

*Ask for more cuts* ends the step **without writing** and reports `action: more`, with the editor's current selection in `proposal.preselected`. A flow can then ask the model again and pass the earlier proposal with `previous`:

```yaml
- step: APPLY_TEXT_CUTS
  mode: review
  cutBy: "{{ flowContext.cutBy }}"
  allowMore: true
  previous: cutResult.proposal # a flowContext path; absent in the first round
  outputs:
    - type: object
      name: cutResult
```

The earlier cuts are re-measured and keep their ids (`c1`, `c2`, …), new cuts continue the numbering, and the editor's selection is kept and topped up. `previous` is a path into the flow context, not a template; if nothing is stored there yet the step starts fresh, and a value that is not a cut proposal fails with `APPLY_TEXT_CUTS: previous is not a cut proposal`. Put the round in a `WHILE` loop as in the [complete example](#complete-section-example).

## Outputs

`preview` writes a **`TextCutProposal`** to `object`:

| Field | Contents |
| --- | --- |
| `kind` | Always `"text_cut_proposal"` |
| `xpath` | Content root the proposal belongs to |
| `cutBy` | Characters to remove |
| `currentChars` | Story length now |
| `charsPerLine` | Line estimate setting (`0` = off) |
| `blocks` | `[{ id, chars }]` in document order |
| `candidates` | The usable cuts (see below) |
| `preselected` | Ids checked for the target |
| `failed` | Refused cuts: `[{ index, op, block, error, hint }]`, `index` being the position in the cut list |
| `siblingCounts` | Internal: block-type counts used by the structure rule |

Each candidate has `id` (`c1`, `c2`, …), `block`, `role`, `op`, `saves`, `priority`, `conflicts` (ids), `before` and `after` (display text without tags), `reason`, plus the resolved `edit` and the highlight `ranges`. A `delete` also carries a `siblingKey`.

`review` and `apply` write a **`TextCutReport`**:

| Field | Contents |
| --- | --- |
| `kind` | Always `"text_cut_report"` |
| `action` | What happened (see below) |
| `cutBy` | Target in characters |
| `selected` | Ids of the selection the report is about |
| `plannedSaving` | Characters the selection removes |
| `missing` | Characters still missing: `max(0, cutBy − plannedSaving)` |
| `lengthBefore` / `lengthAfter` | Story length before and after; measured again after a write, equal when nothing was written |
| `linesAfter` | Estimated lines after the selection, on apply results when `charsPerLine` > 0 |
| `hint` | Why nothing was written, for `failed`, `empty` and `fits` |
| `edits` | The underlying [`TextEditReport`](./story-text-edits.md#outputs) when a write was attempted |
| `proposal` | The proposal; `preselected` holds the selection — feed it to `previous` for the next round |

| `action` | Meaning | Written? |
| --- | --- | --- |
| `applied` | The selection was written. | yes |
| `cancelled` | The editor cancelled, or applied an empty selection. | no |
| `more` | The editor asked for more cuts. | no |
| `fits` | `cutBy` ≤ 0: the story already fits. | no |
| `empty` | The model proposed no usable cut. | no |
| `failed` | The selection was invalid, or the write was refused (see `hint`). | no |

Branch on `action` to tell the editor what happened:

```yaml
- step: SWITCH
  condition: "{{ flowContext.cutResult.action }}"
  cases:
    applied:
      - step: MONITOR_STATUS
        type: success
        message: "Cut {{ flowContext.cutResult.lengthBefore - flowContext.cutResult.lengthAfter }} characters."
    failed:
      - step: MONITOR_STATUS
        type: error
        message: "Nothing was written: {{ flowContext.cutResult.hint | safe }}"
```

## Examples

### Cut in the panel (single round)

```yaml
- step: GET_STORY_BLOCKS
  xpath: /doc/story/text
  showLength: true
  outputs:
    - type: text
      name: blocks
- step: MONITOR_INPUT
  message: "How many characters must be cut?"
  required: true
  outputs:
    - type: text
      name: cutByText
- step: HUB_COMPLETION
  response_format: text_cuts
  behavior: |
    You are a newspaper sub-editor. Propose cuts to shorten the story.
    Never change facts, names, numbers or direct quotes. Prefer removing sentences over rewriting.
    Give every cut a priority and put alternative cuts of the same passage in one group.
  instruction: |
    {{ flowContext.blocks | safe }}

    Propose 5 to 10 cuts that together remove at least {{ (flowContext.cutByText | int) * 2 }} characters.
- step: APPLY_TEXT_CUTS
  mode: review
  xpath: /doc/story/text
  cutBy: "{{ flowContext.cutByText }}"
  charsPerLine: 34
```

### Report the cuts without writing

Works on read-only documents: the preselection is highlighted and the candidates are shown as a table (`MONITOR_TABLE` needs the Flow Monitor).

```yaml
- step: GET_STORY_BLOCKS
  xpath: /doc/story/text
  showLength: true
- step: HUB_COMPLETION
  response_format: text_cuts
  behavior: "You are a newspaper sub-editor. Propose cuts to shorten the story without changing facts."
  instruction: "{{ flowContext.text }}"
- step: APPLY_TEXT_CUTS
  mode: preview
  xpath: /doc/story/text
  cutBy: 800
  outputs:
    - type: object
      name: cuts
- step: SCRIPTING
  script: |
    flowContext.cutRows = flowContext.cuts.candidates.map((cut) => ({
      '#': cut.id,
      Block: cut.block,
      Saves: cut.saves,
      Priority: cut.priority,
      Cut: cut.before,
      Reason: cut.reason,
    }));
    return flowContext;
- step: MONITOR_TABLE
  title: "Cut candidates (preselected: {{ flowContext.cuts.preselected | join(', ') }})"
  data: "{{ flowContext.cutRows }}"
```

Remove the highlights with `CLEAR_TAGS` and `tags: ["mark"]` (or your `markTag`) when the editor is done.

### Cut automatically to a target length

No panel: the preselection is written directly. Use it only where an unattended cut is acceptable.

```yaml
- step: GET_STORY_BLOCKS
  xpath: /doc/story/text
  showLength: true
- step: HUB_COMPLETION
  response_format: text_cuts
  behavior: "You are a newspaper sub-editor. Propose cuts to shorten the story without changing facts."
  instruction: "{{ flowContext.text }}"
- step: APPLY_TEXT_CUTS
  mode: apply
  xpath: /doc/story/text
  targetLength: 4500
  selection: auto
```

## Complete Section Example

The production-ready section: the editor enters the amount in characters or lines, the model proposes cuts in up to three rounds, and an outcome message closes the flow. Values marked `◆ ADAPT` are defined once and reused through YAML anchors.

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/em-al-wi/proactions-schema/main/schema/partial-section.schema.json
section: Cut to Length
context: story
keywords: cut trim shorten overmatter length copyfit print
monitor:
  enabled: true # the cut panel lives in the Flow Monitor
  defaultWidth: 620

actions:
  - id: proactions.text-cut.review
    title: "Cut story to length"
    keywords: cut trim shorten overmatter kürzen
    flow:
      # ── Settings ──────────────────────────────────────────────────────────────────────
      - step: SET
        charsPerLine: &charsPerLine 34 # ◆ ADAPT: average characters per print line
        maxRounds: 3 # ◆ ADAPT: LLM rounds the editor can request in total

      # ── 1 · Read the story ────────────────────────────────────────────────────────────
      - step: GET_STORY_BLOCKS
        monitorLabel: "Reading the story"
        xpath: &xpath /doc/story/text # ◆ ADAPT: root of the print text
        ignoreElements: &ignoreElements [] # ◆ ADAPT: e.g. [factbox, quotebox]
        inlineElements: &inlineElements [b, i, u, em, strong] # ◆ ADAPT: inline tags of your DTD
        showLength: true
        outputs:
          - type: text
            name: blocks
          - type: json
            name: storyLength

      # ── 2 · How much must go? ─────────────────────────────────────────────────────────
      - step: MONITOR_INPUT
        message: >-
          The story has {{ flowContext.storyLength.chars }} characters.
          How much must be cut? Enter characters (e.g. 350) or lines (e.g. 12 lines).
        placeholder: "e.g. 350 or 12 lines"
        required: true
        outputs:
          - type: text
            name: cutByText

      # Accepts "350", "-350", "350 characters", "12 lines" / "12 Zeilen" / "12 lignes" …
      - step: SCRIPTING
        script: |
          const raw = String(flowContext.cutByText || '').trim();
          const number = raw.match(/\d+/);
          const amount = number ? Number(number[0]) : 0;
          const inLines = /(line|zeile|ligne|rig[ah]|l[ií]nea)/i.test(raw);
          flowContext.cutBy = inLines ? amount * flowContext.charsPerLine : amount;
          flowContext.cutRound = 0;
          flowContext.cutAction = 'more';
          return flowContext;

      - step: IF
        condition: "{{ flowContext.cutBy <= 0 }}"
        then:
          - step: MONITOR_STATUS
            type: warning
            message: 'Please enter a number of characters (e.g. 350) or lines (e.g. "12 lines").'
      - step: END_IF
        condition: "{{ flowContext.cutBy <= 0 }}"

      # ── 3 · Propose, choose, apply ────────────────────────────────────────────────────
      - step: WHILE
        condition: "{{ flowContext.cutRound < flowContext.maxRounds and flowContext.cutAction == 'more' }}"
        do:
          - step: SCRIPTING
            script: |
              flowContext.cutRound = flowContext.cutRound + 1;
              // Ask for clearly more than is needed, so the editor has a real choice.
              const pending = flowContext.cutResult ? flowContext.cutResult.missing : flowContext.cutBy;
              flowContext.cutGoal = Math.ceil(1.75 * Math.max(pending, flowContext.cutBy / 2));
              const earlier = flowContext.cutResult ? flowContext.cutResult.proposal.candidates : [];
              flowContext.covered = earlier
                .map((cut) => '- ' + cut.block + ' ' + cut.op + ': ' + cut.before.slice(0, 80))
                .join('\n');
              return flowContext;

          - step: HUB_COMPLETION
            monitorLabel: "Proposing cuts (round {{ flowContext.cutRound }})"
            response_format: text_cuts
            behavior: |
              You are an experienced newspaper sub-editor. The story must be shortened to fit its print
              slot. You propose cuts; the editor decides which of them are applied.

              Editorial rules (◆ ADAPT to your house style):
              - Never change facts, names, numbers, dates or direct quotes. Never add information.
              - Keep the language and the tone of the story; a shortened passage must read naturally.
              - Cut in this order: repetitions and redundant information, decorative detail and colour,
                additional examples, background, secondary reactions and quotes. Keep the core news
                (who, what, when, where, why) and the main quotes.
              - Prefer removing whole sentences or clauses over rewriting. Use "rewrite" only to
                condense a paragraph whose content must stay.
              - Give the editor a real choice: several small and medium cuts (a clause, a sentence, a
                paragraph) spread over the story, rather than a few large ones. Where useful, offer
                alternatives for the same passage, e.g. drop a sentence vs. condense it.

              Format (keep unchanged):
              - Every story line is "[id role · characters] text". Put the id (e.g. "b3") in "block" and
                use the character count to size your cuts.
              - Every cut must stand on its own: the editor may apply any combination of cuts that are not
                alternatives of each other.
              - Two or more cuts of the same passage are alternatives: give them the same "group" label.
                A cut without alternatives has an empty group.
              - priority: 1 = cut first (least important), 2 = normal, 3 = only if needed.
              - op "delete": removes the whole paragraph; "find" and "replace" are empty.
              - op "replace": "find" is copied character by character from the paragraph — typographic
                quotes (“ ” ‘ ’), apostrophes, dashes and inline tags such as <b>…</b> exactly as they appear
                — and occurs only once in it; "replace" is the shorter text. An empty "replace" removes the
                passage: then include one adjacent space in "find" so no double space remains.
              - op "rewrite": "replace" is the complete shortened paragraph, keeping its inline tags;
                "find" is empty.
              - reason: what is removed and why, in a few words and in the language of the story; no
                character counts.
              - Return an empty "cuts" array when nothing can be cut without losing essential content.
            instruction: |
              {{ flowContext.blocks | safe }}

              The story has {{ flowContext.storyLength.chars }} characters and must lose {{ flowContext.cutBy }}.
              Propose 5 to 10 cuts of different sizes that together remove at least {{ flowContext.cutGoal }}
              characters, so the editor can combine them to fit.
              {% if flowContext.covered %}
              These cuts were already proposed. Do not repeat them; propose different ones:
              {{ flowContext.covered | safe }}
              {% endif %}

          - step: APPLY_TEXT_CUTS
            mode: review
            cutBy: "{{ flowContext.cutBy }}"
            charsPerLine: *charsPerLine
            allowMore: true
            previous: cutResult.proposal # merges the earlier rounds (absent in round 1)
            xpath: *xpath
            ignoreElements: *ignoreElements
            inlineElements: *inlineElements
            outputs:
              - type: object
                name: cutResult

          - step: SET
            cutAction: "{{ flowContext.cutResult.action }}"

      # ── 4 · Outcome ───────────────────────────────────────────────────────────────────
      - step: SWITCH
        condition: "{{ flowContext.cutResult.action }}"
        cases:
          applied:
            - step: MONITOR_STATUS
              type: success
              message: >-
                Cut {{ flowContext.cutResult.lengthBefore - flowContext.cutResult.lengthAfter }} characters:
                {{ flowContext.cutResult.lengthBefore }} → {{ flowContext.cutResult.lengthAfter }}.
                {% if flowContext.cutResult.missing > 0 %}Still {{ flowContext.cutResult.missing }} characters
                short of the target — run the action again for more cuts.{% endif %}
          more:
            - step: MONITOR_STATUS
              type: warning
              message: >-
                No cuts were applied after {{ flowContext.maxRounds }} rounds. Run the action again to
                start over.
          cancelled:
            - step: MONITOR_STATUS
              type: info
              message: "Cancelled — the story was not changed."
          empty:
            - step: MONITOR_STATUS
              type: warning
              message: "No usable cuts were proposed: {{ flowContext.cutResult.hint | safe }}"
          failed:
            - step: MONITOR_STATUS
              type: error
              message: "Nothing was written: {{ flowContext.cutResult.hint | safe }}"
```

To take the amount from a copyfit service instead of asking the editor, replace `MONITOR_INPUT` with a step that writes the overmatter (characters or lines) to `flowContext.cutByText`.

## Error Codes

A refused cut never reaches the editor. It is listed in `proposal.failed[]` with an `error` and a `hint`:

| Code | Meaning | What to do |
| --- | --- | --- |
| `not_found` | `find` does not occur in the block. | Tell the model to copy `find` character by character, including typographic quotes and inline tags; pass the blocks with `| safe`. |
| `ambiguous_match` | `find` occurs more than once in the block. | Ask for longer, unique `find` strings. |
| `stale_block` | The block changed since the read, or the steps use different block options. | Re-run `GET_STORY_BLOCKS`; use identical options on both steps. |
| `unknown_block` | The `block` id is not in the read. | Use the ids from `GET_STORY_BLOCKS`. |
| `op_not_allowed` | The op is not `delete`, `replace` or `rewrite`. | The `text_cuts` schema already restricts ops; check custom schemas. |
| `protected_markup` | The cut would remove hidden content (embedded objects, strikeouts, placeholders). | Expected: such paragraphs can be shortened with `replace`/`rewrite`, not deleted. |
| `structure_required` | The delete would remove the only block of its type. | Expected: shorten the block instead. |
| `no_saving` | The cut does not make the story shorter. | Harmless; the model lengthened or only reworded the passage. |
| `unchanged` | The cut changes nothing. | Harmless. |
| `duplicate` | The same cut was already proposed in this or an earlier round. | Harmless; the first one is kept. |
| `too_many_candidates` | More than 60 usable cuts in one round. | Ask for fewer, larger cuts. |
| `escaped_markup`, `replacement_not_well_formed`, `disallowed_elements`, `disallowed_attributes`, `disallowed_node_type` | The replacement is not valid content. | Return plain text plus the inline tags shown in the block view. |

When an applied selection fails, `report.action` is `failed`, `hint` explains why, and `report.edits` holds the [story text edits report](./story-text-edits.md#error-codes) with per-edit errors.

Step-level failures stop the flow instead: `APPLY_TEXT_CUTS: set cutBy or targetLength`, `cutBy must be a number`, `previous is not a cut proposal`, `cuts must be an array or an object with a "cuts" array`, `mode must be one of review, preview, apply`, `Review mode requires the Flow Monitor`, and the read-only error.

## Troubleshooting

- **`Review mode requires the Flow Monitor`** — set `monitor: enabled: true` on the section or action, or use `mode: preview`.
- **`It is not possible to write content in read-only mode!`** — the story is not checked out; use `mode: preview` or `forceWrite: true` for trusted flows.
- **Many `not_found` failures** — the blocks reached the model escaped (`&lt;b&gt;`): use `{{ flowContext.blocks | safe }}` in multi-line instructions. Otherwise the model re-typed quotes or dropped inline tags; ask it to copy `find` character by character.
- **Only a few, large cuts** — ask for a number of cuts of different sizes ("5 to 10 cuts of different sizes"), and for more characters than needed so the editor can combine.
- **The preselection overshoots** — the least important cuts (priority 1) are used first, even when large. Swap them in the panel, or ask the model to rank large cuts as priority 2.
- **The meter disagrees with the layout** — the meter counts the blocks under `xpath`. Point `xpath` at the element the print frame shows (e.g. `/doc/story/text`), and set `charsPerLine` to your column's average.
- **Highlights stay in a Prime story after a preview** — run `CLEAR_TAGS` with `tags: ["mark"]`. The review panel removes its own highlights when it closes.
- **HTTP 400 about `response_format` or `json_schema`** — `text_cuts` uses structured outputs; the model or Hub target must support `json_schema` with `strict: true`.

## See Also

- [`APPLY_TEXT_CUTS`](../../step-library/steps/APPLY_TEXT_CUTS.md)
- [`GET_STORY_BLOCKS`](../../step-library/steps/GET_STORY_BLOCKS.md)
- [Story Text Edits](./story-text-edits.md)
- [Flow Monitor](./flow-monitor.md)
- [AI Integration](./ai-integration.md)
