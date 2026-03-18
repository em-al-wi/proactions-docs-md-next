---
sidebar_position: 1
---

# TAG_PURE_TEXT

Adds tags to pure text content in the document. Tags can highlight or annotate specific text ranges. In Swing, tags can be virtual (temporary) or persistent. In Prime, tags are always persistent.

## At a glance
- **Category** Editor
- **Version:** 1.2.0
- **Applications:** all
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `tags` | Array of tags to add. Each tag must have: tag (tag name), start (position), size (length), and optional attributes (key-value pairs). Can be a hardcoded array, a flowContext variable, or read from the default list input. | None |false| true |None|None|
| `virtual` | Whether tags should be virtual (temporary, not saved). Default is false (persistent tags). Only applies to Swing; Prime always persists tags. | false |false| false |None|None|

## Inputs
| Type | Description | Default | Required | Resolved |
|---|---|:---:|:---:|:---:|
| `list` | Default list input for tags when tags config is not specified. | None | false | false |

## Examples

### Add hardcoded marks
```yaml
- step: TAG_PURE_TEXT
  tags:
    - tag: "mark"
      start: 100
      size: 10
```

### Add persistent tags with attributes
```yaml
- step: TAG_PURE_TEXT
  tags:
    - tag: "mark"
      start: 100
      size: 10
      attributes:
        class: "highlight"
    - tag: "mark"
      start: 20
      size: 15
      attributes:
        class: "important"
```

### Add virtual tags from flowContext
```yaml
- step: TAG_PURE_TEXT
  tags: "{{ flowContext.analyzedTags }}"
  virtual: true
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
