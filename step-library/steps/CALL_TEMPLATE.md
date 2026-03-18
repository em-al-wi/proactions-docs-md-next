---
sidebar_position: 1
---

# CALL_TEMPLATE

Calls a reusable flow by legacy template name (`name`) or scoped reference (`ref`).

## At a glance
- **Category** Control
- **Aliases** CALL_FLOW
- **Version:** 1.0.6
- **Applications:** all
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `name` | The name of the template to call from the TEMPLATES: section. | None |false| true |None|None|
| `ref` | Reference path to a reusable flow (for example global.flows.myFlow or imports.common.flows.myFlow). | None |false| true |None|None|

### Step-Level Validation Rules

**Require At Least One:**
- At least one of: name, ref

## Examples

### Call a template
```yaml
 - step: CALL_TEMPLATE
   name: "myTemplate"
 ```

### Call a scoped flow by reference
```yaml
 - step: CALL_FLOW
   ref: "section.flows.myFlow"
 ```

## Common Patterns

### Dynamic Template Selection

You can use template expressions to dynamically select which template to call at runtime.

```yaml

- step: SET
  templateToRun: "{{ flowContext.userChoice }}"

- step: CALL_TEMPLATE
  name: "{{ flowContext.templateToRun }}"

```

### Scoped Flow Reference

Use explicit scoped refs for reusable flows in BLOCKS/imports namespaces.

```yaml

- step: CALL_FLOW
  ref: "section.flows.normalize"

```

## Common Pitfalls

:::warning Expecting END_IF or BREAK to stop the parent workflow

**Why:** Templates execute in an isolated flow control context. Flow control steps inside a template only affect the template itself.

**Solution:** Use the END step if you need to stop the entire workflow from within a template.
:::

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
