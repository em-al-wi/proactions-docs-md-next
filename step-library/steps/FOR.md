---
sidebar_position: 1
---

# FOR

Executes a set of steps for a fixed number of iterations (numeric) or iterates over an array (items).

## At a glance
- **Category** Control
- **Version:** 1.1.0
- **Applications:** all
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `var` | The name of the variable to set with the current iteration value. | None |true| false |Pattern: `^[a-zA-Z_][a-zA-Z0-9_]*$`<br/>Length:1-∞|None|
| `start` | The starting value of the loop variable (numeric). | None |false| true |None|None|
| `end` | The ending value of the loop variable (numeric, inclusive). | None |false| true |None|None|
| `by` | Step increment for numeric loops (can be negative). If omitted, inferred from start/end (1 or -1). | None |false| true |None|None|
| `items` | An array to iterate over (e.g., an expression producing an array). If provided, numeric start/end are ignored. | None |false| true |None|Conflicts: start, end, by|
| `do` | Steps to execute in each iteration of the loop. | None |false| false |None|None|

### Step-Level Validation Rules

**Require At Least One:**
- At least one of: items, start

**Require Together:**
- All or none: start, end

## Examples

### For numeric loop from 1 to 5
```yaml
 - step: FOR
   var: "i"
   start: 1
   end: 5
   do:
     - step: SET
       iteration: "{{ flowContext.i }}"
 ```

### For-each over items
```yaml
 - step: FOR
   var: "item"
   items: "{{ flowContext.myList }}"
   do:
     - step: SET
       lastItem: "{{ flowContext.item }}"
 ```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
