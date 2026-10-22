---
sidebar_position: 1
---

# WHILE

Repeatedly executes a set of steps while a condition evaluates to true.

## At a glance
- **Category** Control
- **Version:** 1.0.3
- **Applications:** all
- **Scope:** all

## When to Use

Use **WHILE** to repeat steps until a context-dependent condition becomes false or a BREAK occurs.

## Data Flow

Reads flow context values referenced by `condition` or legacy `test` each iteration. Writes from each `do` subflow are preserved.

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `condition` | The condition to evaluate for the loop. Can be a template expression with \{\{ \}\} syntax. | None |false| true |Length:1-∞|None|
| `test` | Legacy alternative to condition. The test expression to evaluate using JavaScript syntax. | None |false| false |Length:1-∞|None|
| `do` | Steps to execute in each iteration of the loop. | None |false| false |None|None|

### Step-Level Validation Rules

**Require Exactly One:**
- Exactly one of: condition, test

## Examples

### While loop with condition
```yaml
 - step: WHILE
   condition: "{{ flowContext.counter < 10 }} "
   do:
     - step: SET
       counter: "{{ flowContext.counter + 1 }}"
     - step: SHOW_NOTIFICATION
       message: "Iteration {{flowContext.counter}}"
 ```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
