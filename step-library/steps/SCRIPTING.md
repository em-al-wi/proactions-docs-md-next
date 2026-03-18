---
sidebar_position: 1
---

# SCRIPTING

Executes inline JavaScript (`script`) or a JS-style template (`template`). Use with caution — only in trusted flows.

## At a glance
- **Category** Utils
- **Version:** 1.0.4
- **Applications:** all
- **Scope:** all

## When to Use


Use **SCRIPTING** when you need to execute custom JavaScript logic within your workflow.

**Use SCRIPTING when:**
- You need complex multi-statement logic (loops, conditionals, variable assignments)
- You want to transform data structures using JavaScript methods
- You need to evaluate expressions and assign results to variables (\`returnAs\`)
- You want to build JSON output from template literals (\`template\` mode)
- You want to reuse centrally defined scripts via \`scriptRef\`
- You're working with AI tool call parameters that need processing

**Don't use SCRIPTING when:**
- You only need simple variable assignment - use **SET** for single expressions
- You need user input - use **FORM** or **PROMPT** for interactive input
- You can accomplish the task with existing declarative steps

**Security Warning:**
SCRIPTING executes arbitrary JavaScript code without sandboxing. Only use in trusted workflows where:
- Configuration files are controlled by trusted developers
- User input is never passed directly to scripts
- You understand the security implications of code execution

**Available Context Variables:**
Scripts have access to these variables:
- \`flowContext\` - Current workflow context (read/write)
- \`ctx\` - Swing/Prime context object (platform-specific)
- \`client\` - ClientAdapter with methods like \`getTextContent()\`, \`getSelectedWord()\`, etc.
- \`cfg\` - Current step configuration
- \`step\` - Current step instance
- \`params\` - Parameters from AI tool calls (when used in function steps)

**Mode Selection:**
- **script mode** - For JavaScript code blocks that return flowContext
- **script + returnAs** - For expressions that assign to a single variable
- **template mode** - For building JSON from template literals (auto-parsed)

**Performance Note:**
Uses JavaScript's Function constructor which has overhead. For high-frequency operations, consider using dedicated steps.
  

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `script` | A JavaScript expression or block to be executed in the step context. If `mode: "async"` the script is treated as async. | None |false| false |None|None|
| `template` | A template string that will be resolved using the step context and then parsed as JSON. Use this when you want to build structured output. | None |false| false |None|None|
| `scriptRef` | Reference path to a reusable script (for example local.scripts.normalizeName or imports.common.scripts.normalizeName). | None |false| true |None|None|
| `mode` | Execution mode for scripts. Use "async" to run the script using the asynchronous evaluator. | None |false| false |None|None|
| `returnAs` | When specified, the script is treated as an expression (no return statement needed) and the result is assigned to the specified property in flowContext. Cannot be used with template mode. | None |false| false |None|None|

### Step-Level Validation Rules

**Require Exactly One:**
- Exactly one of: script, scriptRef, template

## Examples

### Run a synchronous script
```yaml
- step: SCRIPTING
  script: "flowContext.count = (flowContext.count ?? 0) + 1; return flowContext;"
```

### Run an async script
```yaml
- step: SCRIPTING
  mode: "async"
  script: "const res = await someAsyncFunction(cfg, flowContext); flowContext.result = res; return flowContext;"
```

### Run a reusable script via scriptRef
```yaml
AI_KIT:
  SECTIONS:
    - section: Editorial
      actions:
        - title: Normalize Name
          blocks:
            scripts:
              normalizeName: "flowContext.name.toLowerCase()"
          flow:
            - step: SCRIPTING
              scriptRef: local.scripts.normalizeName
              returnAs: normalizedName
```

### Generate structured output from a template
```yaml
- step: SCRIPTING
  template: '{"message": "Hello {{flowContext.userName}}", "items": {{ JSON.stringify(flowContext.items || []) }} }'
```

### Evaluate expression and assign to variable
```yaml
- step: SCRIPTING
  script: |
    params.suggestions
      .map(item => {
        const start = flowContext.originalText.indexOf(item.segment);
        if (start === -1) return null;
        return {
          tag: "mark",
          start: start,
          size: item.segment.length,
          attributes: { class: item.priority === 1 ? "high" : "medium" }
        };
      })
      .filter(Boolean)
  returnAs: tags
```

### Evaluate async expression and assign to variable
```yaml
- step: SCRIPTING
  mode: async
  script: "fetch('https://api.example.com/data').then(res => res.json())"
  returnAs: apiData
```

### Access context variables
```yaml
- step: SCRIPTING
  script: |
    // Access client adapter methods
    const selectedText = client.getSelectedText();
    const documentId = client.getDocumentId();

    // Access flowContext
    const existingData = flowContext.previousResult;

    // Combine and return
    flowContext.metadata = {
      selection: selectedText,
      docId: documentId,
      processedAt: new Date().toISOString(),
      data: existingData
    };
    return flowContext;
```

### Process AI tool call parameters
```yaml
# In an AI tool call function
tools:
  - name: highlightImportantSections
    description: Highlight important text sections
    parameters:
      type: object
      properties:
        sections:
          type: array
          items:
            type: object
            properties:
              text:
                type: string
              importance:
                type: string
                enum: [high, medium, low]
    steps:
      - step: SCRIPTING
        script: |
          // params contains the AI's function call arguments
          params.sections
            .filter(s => s.importance === 'high')
            .map(s => ({
              text: s.text,
              position: flowContext.originalText.indexOf(s.text)
            }))
        returnAs: highlights
```

### Complex multi-step logic
```yaml
- step: SCRIPTING
  script: |
    // Multi-statement script with loops and conditionals
    const items = flowContext.items || [];
    const results = [];

    for (const item of items) {
      if (item.score > 0.8) {
        results.push({
          id: item.id,
          label: 'high-confidence',
          processed: true
        });
      } else if (item.score > 0.5) {
        results.push({
          id: item.id,
          label: 'medium-confidence',
          needsReview: true
        });
      }
    }

    flowContext.processedItems = results;
    flowContext.totalProcessed = results.length;
    flowContext.timestamp = Date.now();

    return flowContext;
```

## Common Pitfalls

:::warning Don't forget to return flowContext in normal script mode

**Why:** Scripts must return the flowContext object (or a modified version) to continue the workflow. Forgetting this breaks the flow.

**Solution:** Always end scripts with `return flowContext;` unless using `returnAs` mode. Use `returnAs` for simple expressions to avoid boilerplate.
:::

:::warning Don't mix template syntax with script syntax

**Why:** Template mode uses `$\{\}` for interpolation and expects JSON output. Script mode uses pure JavaScript. Mixing them causes errors.

**Solution:** Use `template` for building JSON with interpolation. Use `script` for JavaScript logic. Never use both in the same step.
:::

:::warning Don't forget await for async operations with returnAs

**Why:** When using `returnAs` with async expressions, you must set `mode: "async"` or promises won't be resolved.

**Solution:** Use `mode: "async"` with `returnAs` for any expression that returns a Promise (fetch, async functions, etc.).
:::

:::warning Don't pass user input directly to scripts

**Why:** User input in scripts can lead to code injection vulnerabilities. SCRIPTING has no sandboxing.

**Solution:** Validate and sanitize all user input before using in scripts. Better yet, use declarative steps instead of SCRIPTING for user data.
:::

:::warning Don't expect flowContext mutations in returnAs mode to persist

**Why:** When using `returnAs`, only the specified property is set. Direct mutations to flowContext are ignored.

**Solution:** Use normal script mode if you need to mutate multiple flowContext properties. Use `returnAs` only for single-value assignments.
:::

## See Also

**Related Steps:**
- **[SET](SET.md)** - Alternative for simple expressions: Use SET for simple value assignments and single expressions. Use SCRIPTING for multi-line logic with loops and conditionals.
- **[IF](IF.md)** - Use together for conditional logic: Use IF for flow control based on conditions. Use SCRIPTING inside IF blocks for complex conditional logic.
- **[FOR](FOR.md)** - Use together for iteration: Use FOR to iterate over arrays. Use SCRIPTING inside FOR loops for complex item processing.

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
