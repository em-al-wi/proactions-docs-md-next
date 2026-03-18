---
sidebar_position: 1
---

# PARALLEL

Runs multiple step lists in parallel and merges their flow contexts upon completion.

## At a glance
- **Category** Control
- **Version:** 1.0.0
- **Applications:** all
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `steps` | An array of step arrays, where each sub-array represents a branch of steps to execute in parallel. | None |true| false |Items:2-∞|None|

## Examples

### Run two branches in parallel
```yaml
 - step: PARALLEL
   steps:
     - # Branch 1
       - step: SET
         branch1Var: "value1"
     - # Branch 2
       - step: SET
         branch2Var: "value2"
 ```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
