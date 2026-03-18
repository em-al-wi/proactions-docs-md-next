---
sidebar_position: 1
---

# EDAPI_OBJECT_CONTENT

Fetch object content from EDAPI and return it as blob or configured outputs. The content id can be provided via cfg.contentId or via a text input.

## At a glance
- **Category** EDAPI
- **Version:** 1.0.4
- **Applications:** all
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `contentId` | Explicit content id to fetch. If omitted, the step will try to read the id from the default text input. | None |false| true |None|None|
| `format` | Format of the content to fetch. | lowres |false| false |None|None|

## Inputs
| Type | Description | Default | Required | Resolved |
|---|---|:---:|:---:|:---:|
| `text` | Alternate input path to read the content id from the flow context (used if contentId is not provided). | None | false | false |

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `blob` | The fetched content blob (default output). | false |

## Examples

### Fetch content by id from flowContext
```yaml
- step: SELECTED_OBJECT
- step: EDAPI_OBJECT_CONTENT
  # contentId: "{{ flowContext.text }}"
  format: lowres
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
