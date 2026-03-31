---
sidebar_position: 1
---

# INSERT_TEMPLATE_SECTION

Creates a new section in the document using the specified template path, relative to the currently elected item

## At a glance
- **Category** Browser

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `templatePath` | The path to the template to be inserted (supports variables) | None |true| false |None|None|
| `insertAfter` | Whether the section should be inserted after the current section (true) or before (false). Defaults to true | None |false| false |None|None|

## Examples

### Insert a new section based on a template
```yaml
- step: INSERT_TEMPLATE_SECTION
  templatePath: '/Content/Templates/My Awesome Template'
  insertAfter: true
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
