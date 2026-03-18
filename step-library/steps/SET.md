---
sidebar_position: 1
---

# SET

Sets variables in the flow context. Can execute steps, evaluate expressions, or set key-value pairs.

## At a glance
- **Category** Control
- **Version:** 1.0.3
- **Applications:** all
- **Scope:** all

## When to Use


Use **SET** when you need to create or modify variables in the workflow context for use in subsequent steps.

**Use SET when:**
- You need to store simple values (strings, numbers, booleans, objects)
- You want to prepare data before passing to other steps
- You need to resolve templates and store the result
- You want to execute JavaScript expressions and store results
- You need to transform or combine existing variables

**Don't use SET when:**
- You need user input - use **FORM** or **PROMPT** for interactive input
- You need complex multi-statement logic - use **SCRIPTING** for full JavaScript programs
- You only need to pass data without storing - most steps can read flowContext directly

**Alternatives:**
- **SCRIPTING** - For complex JavaScript logic with multiple statements and control flow
- **FORM** - For collecting user input interactively
- **PROMPT** - For simple single-value user input
  

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `steps` | Steps to execute before setting the variable. The result is stored in the flow context. | None |false| false |None|None|
| `expression` | JavaScript expression to evaluate and set as the variable value. | None |false| false |None|None|
| `text` | Text to resolve (with variables) and set as the variable value. | None |false| true |None|None|
| `raw_text` | Raw text to set as the variable value without resolution. | None |false| false |None|None|
| `name` | Name of the variable to set in the flow context. | None |false| false |None|None|
| `convertTo` | Convert the value to a specific type (e.g., "number"). | None |false| false |None|None|

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `text` | The resolved value stored in the flow context (when using legacy name/text syntax). | false |

## Examples

### Set multiple variables (new syntax)
```yaml
 - step: SET
   userId: "{{ flowContext.currentUser.id }}"
   isActive: true
   count: 5
   convertTo: "number"
 ```

### Set a variable using text (legacy syntax)
```yaml
 - step: SET
   name: "userMessage"
   text: "Hello {{userName}}"
 ```

### Set numeric variables
```yaml
 - step: SET
   count: 5
   convertTo: "number"
 ```

## Common Pitfalls

:::warning Don't confuse new syntax with legacy syntax - mixing them causes unexpected behavior

**Why:** SET supports two different syntaxes: direct properties (new) and name/text (legacy). Mixing them can create unintended variables or overwrite data.

**Solution:** Use new syntax for most cases (simpler): `SET userId: "123"`. Use legacy syntax only when needed for compatibility: `SET name: "userId" text: "123"`.
:::

:::warning Don't forget that template strings are resolved - use raw_text for literal strings

**Why:** The `text` property resolves \{\{ \}\} templates. If your string contains \{\{ or \}\}, it will be interpreted as a template.

**Solution:** Use `raw_text` for literal strings that contain \{\{ or \}\} characters, or escape them in templates.
:::

:::warning Don't forget type conversion - numbers in templates are strings

**Why:** Template resolution produces strings. "\{\{ 5 \}\}" becomes "5" (string), not 5 (number), which breaks numeric operations.

**Solution:** Use `convertTo: "number"` to convert string values to numbers, or use expression mode for numeric calculations.
:::

## See Also

**Related Steps:**
- **[SCRIPTING](SCRIPTING.md)** - Alternative for complex logic: Use SCRIPTING for multi-line JavaScript with loops and conditionals. Use SET for simple value assignments and single expressions.
- **[FORM](FORM.md)** - Alternative for user input: Use FORM to collect data from users interactively. Use SET to store programmatic values.
- **[PROMPT](PROMPT.md)** - Alternative for simple input: Use PROMPT for single user input. Use SET for storing calculated or static values.
- **[FOR](FOR.md)** - Use together for iteration: Use SET inside FOR loops to accumulate results or maintain counters.
- **[IF](IF.md)** - Use together for conditional assignment: Wrap SET in IF to conditionally set variables based on logic.

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
