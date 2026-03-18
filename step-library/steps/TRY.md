---
sidebar_position: 1
---

# TRY

Executes a set of steps in a try block, and if an error occurs, executes catch steps.

## At a glance
- **Category** Control
- **Version:** 1.1.0
- **Applications:** all
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `try` | Steps to execute in the try block. | None |true| false |None|None|
| `catch` | Steps to execute if an error occurs in the try block. | None |false| false |None|None|

## Examples

### Try catch example
```yaml
 - step: TRY
   try:
     - step: SET
       value: "{{ undefinedVar }}"
   catch:
     - step: SET
       errorHandled: true
 ```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
