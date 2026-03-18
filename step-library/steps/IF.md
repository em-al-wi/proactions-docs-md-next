---
sidebar_position: 1
---

# IF

Execute a conditional branch based on a condition

## At a glance
- **Category** Control
- **Version:** 1.0.3
- **Applications:** all
- **Scope:** all

## When to Use


Use **IF** when you need to execute different steps based on a true/false condition.

**Use IF when:**
- You have a simple true/false decision to make
- You need to execute different workflows based on a condition
- You want to skip steps conditionally
- You need to validate data before proceeding
- You want to handle different cases in your workflow

**Don't use IF when:**
- You have multiple distinct values to match against - use **SWITCH** instead (cleaner for 3+ options)
- You need complex boolean logic with many conditions - consider **SCRIPTING** for readability
- You only need to check if a variable exists - template syntax \{\{ var \}\} handles this

**Alternatives:**
- **SWITCH** - Better for matching one value against multiple specific cases
- **SCRIPTING** - Better for complex conditional logic with multiple variables
- **TRY/catch** - Better for error handling scenarios
  

## Prerequisites

**Required Context:**
- Condition can reference any flowContext variables
- Template syntax \{\{ \}\} evaluates to JavaScript expressions

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `condition` | The condition to evaluate. Can be a template expression with \{\{ \}\} syntax. | None |false| true |None|None|
| `test` | Legacy alternative to condition. The condition to evaluate using JavaScript syntax. | None |false| false |None|None|
| `then` | Steps to execute if the condition is true | None |false| false |None|None|
| `else` | Steps to execute if the condition is false | None |false| false |None|None|

### Step-Level Validation Rules

**Require Exactly One:**
- Exactly one of: condition, test

## Examples

### Basic conditional execution
```yaml
 - step: IF
   condition: "{{ userInput === 'yes' }}"
   then:
     - step: SHOW_NOTIFICATION
       message: "Proceeding..."
 ```

## See Also

**Related Steps:**
- **[SWITCH](SWITCH.md)** - Alternative for multi-value matching: Use SWITCH when checking one variable against 3+ specific values. Use IF for true/false or 2-value conditions.
- **[SCRIPTING](SCRIPTING.md)** - Alternative for complex logic: Use SCRIPTING for complex boolean logic with many variables. Use IF for simple condition checks.
- **[SET](SET.md)** - Use together for conditional assignment: Wrap SET in IF to conditionally set variables.

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
