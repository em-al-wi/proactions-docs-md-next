# Reusable Blocks

Reusable Blocks let you define shared constants, computed variables, flow fragments, and script snippets once and reference them by name anywhere in your configuration. Use them across sections, actions, and external module files.

Without blocks, you end up repeating the same model name, API base path, or cleanup step sequence in dozens of actions. Blocks give those values a single, named home.

## Block Types

Every scope level supports the same four sub-keys inside a `blocks:` entry:

```yaml
blocks:
  constants:     # Static values — strings, numbers, booleans
    apiBase: /SysConfig/ProActions
    defaultModel: gpt-5.4

  vars:          # Lazy Nunjucks expressions — re-evaluated at point-of-use
    userLabel: "{{ flowContext.userName | default('User') }}"
    endpoint: "{{ global.constants.apiBase }}/translate"

  flows:         # Step arrays — callable via CALL_FLOW ref:
    onError:
      - step: SHOW_NOTIFICATION
        notificationType: error
        message: "Error: {{ flowContext.error }}"

  scripts:       # JS strings — callable via SCRIPTING scriptRef:
    stripTags: "flowContext.text.replace(/<[^>]*>/g, '')"
    countWords: "flowContext.text.split(/\\s+/).filter(Boolean).length"
```

### `constants` vs `vars` — which to use?

This is the most important design choice when writing blocks:

| | `constants` | `vars` |
|---|---|---|
| **Value type** | Raw YAML value (string, number, boolean) | Nunjucks template string |
| **Nunjucks in value** | No - treated as a literal | Yes - fully evaluated |
| **When resolved** | Once, at config/scope-build time | Every time the var is accessed |
| **Can read `flowContext`** | No | Yes |
| **Can reference other scopes** | No | Yes (`{{ global.constants.x }}`) |
| **Best for** | Paths, model names, language codes, static flags | Derived labels, dynamic endpoints, values that depend on flow state |

**Use `constants`** for anything that never changes between executions — file paths, API endpoint bases, model names, default language codes.

**Use `vars`** when the value depends on another variable or on the current flow state. Because vars are lazy, they are re-evaluated fresh each time they are accessed, so a var can return different values within the same flow if `flowContext` changes between steps.

```yaml
blocks:
  constants:
    # Fine as a constant — never changes
    targetLang: de

  vars:
    # Must be a var — depends on flowContext at runtime
    headline: "{{ flowContext.text | truncate(80) }}"
    # Can reference constants from any scope
    translationEndpoint: "{{ global.constants.apiBase }}/v2/translate"
```

:::note Circular `vars` references
A `vars` entry that references another `vars` entry that eventually references the first one will throw a deterministic error at runtime: `Circular vars reference detected: a → b → a`. Reference `constants` (not other `vars`) to avoid this.
:::

---

## Scope Levels

Blocks are defined at three levels in your configuration. Each level gets a distinct namespace in templates and step options:

| Namespace | Defined in | Available to |
|---|---|---|
| `global` | `AI_KIT.BLOCKS` | All sections and actions |
| `section` | `SECTIONS[n].blocks` | All actions in that section |
| `local` | `actions[n].blocks` | That action only |
| `imports.<alias>` | External module file | See [Module Imports](#module-imports) |

### Global blocks — `AI_KIT.BLOCKS`

```yaml
AI_KIT:
  BLOCKS:
    constants:
      apiBase: /SysConfig/ProActions
      defaultModel: gpt-4o
    flows:
      onError:
        - step: SHOW_NOTIFICATION
          notificationType: error
          message: "{{ flowContext.error }}"
    scripts:
      slugify: "flowContext.text.toLowerCase().replace(/\\s+/g, '-')"
```

Access anywhere as `{{ global.constants.apiBase }}`, `CALL_FLOW ref: global.flows.onError`, etc.

### Section blocks — `section.blocks`

```yaml
AI_KIT:
  SECTIONS:
    - section: Editorial
      blocks:
        constants:
          sectionLabel: Editorial
        vars:
          # Can reference global constants
          header: "{{ section.constants.sectionLabel }} — {{ global.vars.userDisplayName }}"
        flows:
          cleanup:
            - step: SET
              text: "{{ flowContext.text | trim }}"
        scripts:
          countWords: "flowContext.text.split(/\\s+/).filter(Boolean).length"
      actions:
        # ...
```

Access within any action of this section as `{{ section.constants.sectionLabel }}`, `CALL_FLOW ref: section.flows.cleanup`, etc.

### Action blocks — `local.*`

```yaml
      actions:
        - title: Translate Article
          blocks:
            constants:
              targetLang: de
            vars:
              snippet: "{{ flowContext.text | truncate(100) }}"
            flows:
              validate:
                - step: IF
                  condition: "{{ flowContext.text | length < 5 }}"
                  then:
                    - step: SHOW_NOTIFICATION
                      message: Too short
            scripts:
              normalize: "flowContext.text.toLowerCase().trim()"
          flow:
            - step: CALL_FLOW
              ref: local.flows.validate
            - step: SCRIPTING
              scriptRef: local.scripts.normalize
              returnAs: normalizedText
```

Access within this action only as `{{ local.constants.targetLang }}`, `CALL_FLOW ref: local.flows.validate`, etc.

---

## Module Imports

Imports let you pull blocks from a separate YAML file and use them under a named alias. This is the main way to share blocks across multiple sections or configuration files.

### Declaring imports

`imports:` can appear at the section level (alias available to all actions in the section) or at the action level (alias available only to that action):

```yaml
AI_KIT:
  SECTIONS:
    - section: Editorial
      # Section-level: alias is available to all actions in this section
      imports:
        - from: /SysConfig/ProActions/lib/common.ai-kit.module.yaml
          as: common
        - from: ./regional.ai-kit.module.yaml   # relative to this file
          as: regional
        - from: "{{ global.constants.apiBase }}/translate.ai-kit.module.yaml"  # dynamic path
          as: translate
      actions:
        - title: Translate
          # Action-level: alias available only to this action
          imports:
            - from: ./deepl-helpers.ai-kit.module.yaml
              as: deepl
          flow:
            - step: CALL_FLOW
              ref: imports.translate.flows.toGerman
            - step: SCRIPTING
              scriptRef: imports.deepl.scripts.formatResult
              returnAs: result
```

### `from` path formats

| Format | Example | Resolved from |
|---|---|---|
| Absolute CMS path | `/SysConfig/ProActions/lib/common.ai-kit.module.yaml` | CMS content root |
| Relative path | `./shared/common.ai-kit.module.yaml` | The file that declares the import |
| Nunjucks template | `"{{ global.constants.apiBase }}/module.yaml"` | Resolved first, then fetched |

Relative paths (`./` or `../`) resolve from the file that contains the `imports:` declaration — not from the root config. This means included section files (via `!include`) have relative imports that work relative to where that section file lives in the CMS.

### Module file format

A module file is a plain YAML file with a `blocks:` (or `BLOCKS:`) key at the root. It supports all four block types and can itself declare `imports:` (fully recursive):

```yaml
# /SysConfig/ProActions/lib/common.ai-kit.module.yaml

blocks:
  constants:
    sharedModel: gpt-4o-mini
    maxRetries: 3

  vars:
    currentDate: "{{ flowContext.date | default('unknown') }}"

  flows:
    standardCleanup:
      - step: SET
        text: "{{ flowContext.text | trim }}"
      - step: SANITIZE

  scripts:
    slugify: "flowContext.title.toLowerCase().replace(/\\s+/g, '-')"

# A module can itself import from other modules
imports:
  - from: ./base-prompts.ai-kit.module.yaml
    as: prompts
```

### Accessing imported blocks

Use `imports.<alias>.*` in templates, `CALL_FLOW ref:`, and `SCRIPTING scriptRef:`:

```yaml
flow:
  # Templates
  - step: SET
    model: "{{ imports.common.constants.sharedModel }}"
    label: "{{ imports.common.vars.currentDate }}"
    prompt: "{{ imports.prompts.constants.systemPrompt }}"

  # Call a flow from the module
  - step: CALL_FLOW
    ref: imports.common.flows.standardCleanup

  # Use a script from the module
  - step: SCRIPTING
    scriptRef: imports.common.scripts.slugify
    returnAs: slug
```

### Import merge rules

When both section and action declare `imports:`, they are merged at runtime. The action's imports are appended after the section's. If both declare an alias with the same name, the action's alias overwrites the section's (last writer wins).

---

## Using Blocks in Flows

### In any step field — template expressions

Use `{{ namespace.type.key }}` in any step configuration value that supports variable resolution:

```yaml
- step: HUB_COMPLETION
  service: "{{ global.constants.preferredService }}"
  instruction: "Translate to {{ local.constants.targetLang }}: {selectedText}"
  model: "{{ imports.common.constants.defaultModel }}"

- step: IF
  condition: "{{ section.vars.featureEnabled }}"
  then:
    - step: SHOW_NOTIFICATION
      message: "Hello, {{ global.vars.userDisplayName }}"
```

### Calling flows — `CALL_FLOW ref:`

```yaml
# Call a global flow
- step: CALL_FLOW
  ref: global.flows.onError

# Call a section flow
- step: CALL_FLOW
  ref: section.flows.cleanup

# Call a local flow
- step: CALL_FLOW
  ref: local.flows.validate

# Call a flow from an imported module
- step: CALL_FLOW
  ref: imports.common.flows.standardCleanup

# Dynamic ref — resolved at runtime from flowContext
- step: CALL_FLOW
  ref: "{{ flowContext.selectedFlow }}"   # e.g. resolves to "section.flows.postProcess"
```

:::tip Flow isolation
A called flow runs in an isolated flow-control context. `BREAK` and `END_IF` inside a called flow only affect that flow — they do not propagate to the caller. Use the `END` step to stop the entire workflow from within a called flow.
:::

### Running scripts — `SCRIPTING scriptRef:`

```yaml
# All scope levels work the same way
- step: SCRIPTING
  scriptRef: local.scripts.normalize
  returnAs: normalizedText

- step: SCRIPTING
  scriptRef: section.scripts.countWords
  returnAs: wordCount

- step: SCRIPTING
  scriptRef: global.scripts.stripTags
  returnAs: plainText

- step: SCRIPTING
  scriptRef: imports.common.scripts.slugify
  returnAs: slug

# Async script from a module
- step: SCRIPTING
  scriptRef: imports.api.scripts.fetchData
  mode: async
  returnAs: apiResult

# Dynamic scriptRef — resolved at runtime
- step: SCRIPTING
  scriptRef: "{{ flowContext.scriptPath }}"
  returnAs: result
```

A `scriptRef` and an inline `script:` (or `template:`) are mutually exclusive — use exactly one. `returnAs` cannot be combined with `template:`.

---

## All Scope Paths — Reference

The following access paths are valid in any `{{ }}` template expression, `CALL_FLOW ref:`, or `SCRIPTING scriptRef:`:

| Path | Type | Notes |
|---|---|---|
| `global.constants.<key>` | Any YAML value | Defined in `AI_KIT.BLOCKS.constants` |
| `global.vars.<key>` | String (lazy) | Defined in `AI_KIT.BLOCKS.vars` |
| `global.flows.<name>` | Step array | Used with `CALL_FLOW ref:` |
| `global.scripts.<name>` | JS string | Used with `SCRIPTING scriptRef:` |
| `section.constants.<key>` | Any YAML value | Defined in `SECTIONS[n].blocks.constants` |
| `section.vars.<key>` | String (lazy) | Defined in `SECTIONS[n].blocks.vars` |
| `section.flows.<name>` | Step array | Used with `CALL_FLOW ref:` |
| `section.scripts.<name>` | JS string | Used with `SCRIPTING scriptRef:` |
| `local.constants.<key>` | Any YAML value | Defined in `actions[n].blocks.constants` |
| `local.vars.<key>` | String (lazy) | Defined in `actions[n].blocks.vars` |
| `local.flows.<name>` | Step array | Used with `CALL_FLOW ref:` |
| `local.scripts.<name>` | JS string | Used with `SCRIPTING scriptRef:` |
| `imports.<alias>.constants.<key>` | Any YAML value | From an imported module file |
| `imports.<alias>.vars.<key>` | String (lazy) | From an imported module file |
| `imports.<alias>.flows.<name>` | Step array | Used with `CALL_FLOW ref:` |
| `imports.<alias>.scripts.<name>` | JS string | Used with `SCRIPTING scriptRef:` |

All four namespaces are available simultaneously. A `local.*` value takes no automatic precedence — all namespaces coexist and you must always use the fully-qualified path.

---

## Error Reference

| Situation | Severity | What happens |
|---|---|---|
| Two imports with the same `as` alias in the **same scope** | **Fatal** | Config fails to load: `Import alias collision: '...' is already defined in this scope` |
| Circular import chain (`A → B → A`) | **Fatal** | Config fails to load: `Import cycle detected: A → B → A` |
| `!include` target file not found | **Non-fatal** | Skipped silently; key gets a safe empty fallback; warning logged to console |
| `imports[].from` file not found | **Non-fatal** | Alias is omitted; warning logged to console; other imports still load |
| `CALL_FLOW ref:` path doesn't exist in scope | Runtime error | Flow stops: `No flow found for ref '...'` |
| `CALL_FLOW` call depth exceeds 64 | Runtime error | Flow stops: `CALL_FLOW max call depth exceeded` (prevents infinite recursion) |
| `SCRIPTING scriptRef:` doesn't resolve to a string | Runtime error | Flow stops: `scriptRef '...' does not resolve to a script string` |
| `script:` and `scriptRef:` both set | Runtime error | Mutually exclusive — exactly one is allowed |
| Circular `vars` reference (`a → b → a`) | Runtime error | Flow stops with full cycle path in the error message |

The distinction between fatal and non-fatal is important: alias collisions and import cycles prevent the configuration from loading at all, while missing files are silently skipped so that a single bad `!include` or unreachable module file does not break the entire configuration.

---

## Complete Example

A configuration with all four block types at all three scope levels and a module import:

```yaml
AI_KIT:
  SERVICES:
    openai:
      type: completion
      apiKey: sk-...

  # ── Global blocks ─────────────────────────────────────────────────────────
  BLOCKS:
    constants:
      apiBase: /SysConfig/ProActions
      defaultModel: gpt-4o
    vars:
      # Lazy: re-evaluated on every access against current flowContext
      userLabel: "{{ flowContext.userName | default('User') }}"
    flows:
      onError:
        - step: SHOW_NOTIFICATION
          notificationType: error
          message: "Error: {{ flowContext.error }}"
    scripts:
      stripTags: "flowContext.text.replace(/<[^>]*>/g, '')"

  SECTIONS:
    - section: Editorial
      # ── Section-level imports ──────────────────────────────────────────────
      imports:
        - from: "{{ global.constants.apiBase }}/lib/common.ai-kit.module.yaml"
          as: common
        - from: ./translations.ai-kit.module.yaml   # relative to this config file
          as: i18n

      # ── Section blocks ─────────────────────────────────────────────────────
      blocks:
        constants:
          sectionLabel: Editorial
        vars:
          sectionHeader: "{{ section.constants.sectionLabel }} — {{ global.vars.userLabel }}"
        flows:
          cleanup:
            - step: SET
              text: "{{ flowContext.text | trim }}"
        scripts:
          countWords: "flowContext.text.split(/\\s+/).filter(Boolean).length"

      actions:
        - title: Translate and Clean
          # ── Local (action-level) blocks ────────────────────────────────────
          blocks:
            constants:
              targetLang: de
            vars:
              snippet: "{{ flowContext.text | truncate(100) }}"
            flows:
              validate:
                - step: IF
                  condition: "{{ flowContext.text.length < 5 }}"
                  then:
                    - step: SHOW_NOTIFICATION
                      message: Text is too short
            scripts:
              normalize: "flowContext.text.toLowerCase().trim()"

          flow:
            # Use constants and vars in step fields
            - step: SET
              lang: "{{ local.constants.targetLang }}"
              model: "{{ global.constants.defaultModel }}"
              header: "{{ section.vars.sectionHeader }}"

            # Call local, section, and imported flows
            - step: CALL_FLOW
              ref: local.flows.validate
            - step: CALL_FLOW
              ref: section.flows.cleanup
            - step: CALL_FLOW
              ref: imports.common.flows.standardHeader

            # Run scripts from different scopes
            - step: SCRIPTING
              scriptRef: local.scripts.normalize
              returnAs: normalizedText
            - step: SCRIPTING
              scriptRef: section.scripts.countWords
              returnAs: wordCount
            - step: SCRIPTING
              scriptRef: global.scripts.stripTags
              returnAs: plainText
            - step: SCRIPTING
              scriptRef: imports.common.scripts.slugify
              returnAs: slug

            # Use imported constants and vars
            - step: HUB_COMPLETION
              instruction: "{{ imports.i18n.vars.translatePrompt }}: {selectedText}"
              model: "{{ imports.common.constants.defaultModel }}"

          # Error handling using a global flow
          onError:
            - step: CALL_FLOW
              ref: global.flows.onError
```

---

## Related Documentation

- [Configuration Basics](./basics.md) — full configuration structure and action attributes
- [Workflow Design](./workflow-design.md) — patterns for using blocks effectively, migration from `TEMPLATES`
- [CALL_TEMPLATE / CALL_FLOW step](../../step-library/steps/CALL_TEMPLATE.md) — full step reference
- [SCRIPTING step](../../step-library/steps/SCRIPTING.md) — `scriptRef`, `returnAs`, and `mode` options
- [Working with Variables](../../getting-started/working-with-variables.md) — full template syntax reference
