---
sidebar_position: 1
---

# BREAK

Breaks out of the current loop (e.g., WHILE or FOR) based on the given condition.

## At a glance
- **Category** Control
- **Version:** 1.1.0
- **Applications:** all
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `condition` | The condition to evaluate. Loop will be broken when condition evaluates to true. If not provided, breaks unconditionally. | None |false| true |None|None|

## Examples

### Break loop if condition met
```yaml
 - step: WHILE
   condition: "{{ flowContext.counter < 10 }}"
   do:
     - step: SET
       counter: "{{ flowContext.counter + 1 }}"
       convertTo: "number"
     - step: BREAK
       condition: "{{ flowContext.counter == 5 }}"
 ```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
