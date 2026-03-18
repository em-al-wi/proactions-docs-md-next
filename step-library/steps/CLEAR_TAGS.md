---
sidebar_position: 1
---

# CLEAR_TAGS

Removes all occurrences of the specified tags from the document or report while preserving their text content. This is useful for cleaning up temporary markup or removing specific tag types.

## At a glance
- **Category** Editor
- **Version:** 1.2.0
- **Applications:** all
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `tags` | Array of tag names to clear (e.g., ["mark", "highlight"]). Can be a hardcoded array, a flowContext variable, or read from the default list input. | None |false| true |None|None|

## Inputs
| Type | Description | Default | Required | Resolved |
|---|---|:---:|:---:|:---:|
| `list` | Default list input for tag names when tags config is not specified. | None | false | false |

## Examples

### Clear specific tag types (hardcoded)
```yaml
- step: CLEAR_TAGS
  tags: ["mark", "highlight"]
```

### Clear tags from flowContext
```yaml
- step: CLEAR_TAGS
  tags: "{{ flowContext.tagsToRemove }}"
```

### Clean up after analysis
```yaml
# First, tag content for analysis
- step: TAG_PURE_TEXT
  tags: "{{ flowContext.analyzedSections }}"
  virtual: true

# Process the tagged content
- step: OPENAI_COMPLETION
  prompt: "Analyze the marked sections..."

# Clean up the temporary tags
- step: CLEAR_TAGS
  tags: ["mark"]
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
