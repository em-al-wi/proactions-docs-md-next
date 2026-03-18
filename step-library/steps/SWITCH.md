---
sidebar_position: 1
---

# SWITCH

Evaluates a switch value and executes the steps for the matching case. Supports both array and object syntax for cases.

## At a glance
- **Category** Control
- **Version:** 1.0.3
- **Applications:** all
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `condition` | The condition to evaluate for the switch value. Can be a template expression with \{\{ \}\} syntax. | None |false| true |Length:1-∞|None|
| `switch` | Legacy alternative to condition. The switch value to evaluate using JavaScript syntax. | None |false| false |Length:1-∞|None|
| `cases` | The cases to match against the switch value. Each case key should match a possible switch value, with the value being an array of steps to execute. Use "default" as a fallback case. | None |true| false |None|None|

### `cases` Object Structure

The cases to match against the switch value. Each case key should match a possible switch value, with the value being an array of steps to execute. Use "default" as a fallback case.

**Dynamic Properties:**

This object accepts dynamic keys. Each key-value pair should follow this structure:
- **Key**: Any string matching the switch value
- **Value Type**: `stepArray`
- **Value Description**: Array of steps to execute when this case matches the switch value
- **Items**: Array of workflow steps

**Special Keys:**
- **`default`** (`stepArray`): Fallback steps to execute when no other case matches

**Object Constraints:**

- **Property Count**:1-∞ properties

### Step-Level Validation Rules

**Require Exactly One:**
- Exactly one of: condition, switch

## Examples

### Switch with object syntax
```yaml
 - step: SWITCH
   condition: "{{ flowContext.userRole }}"
   cases:
     "admin":
       - step: SHOW_NOTIFICATION
         message: "Welcome Admin"
     "user":
       - step: SHOW_NOTIFICATION
         message: "Welcome User"
     "default":
       - step: SHOW_NOTIFICATION
         message: "Welcome Guest"
 ```

### Switch with button selection
```yaml
 - step: FORM
   form:
     selectedAction:
       type: 'select'
       label: 'Choose action'
       options:
         - 'translate'
         - 'summarize'
         - 'improve'
   buttons:
     - type: 'submit'

 - step: SWITCH
   condition: "{{ flowContext.selectedAction }}"
   cases:
     "translate":
       - step: DEEPL_TRANSLATE
         instruction: "{textContent}"
         target_lang: "FR"
     "summarize":
       - step: HUB_COMPLETION
         instruction: "Summarize: {textContent}"
     "improve":
       - step: HUB_COMPLETION
         instruction: "Improve: {textContent}"
     "default":
       - step: SHOW_NOTIFICATION
         message: "No action selected"
 ```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
