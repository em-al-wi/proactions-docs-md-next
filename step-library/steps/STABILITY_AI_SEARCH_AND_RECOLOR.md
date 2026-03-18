---
sidebar_position: 1
---

# STABILITY_AI_SEARCH_AND_RECOLOR

Search and recolor elements in an image using StabilityAI. Provide select_prompt and prompt to find and recolor elements.

## At a glance
- **Category** LLM
- **Aliases** SERVICE_STABILITY_AI_SEARCH_AND_RECOLOR
- **Version:** 1.0.4
- **Applications:** all
- **Scope:** all
- **Default Service:** STABILITY_AI

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `select_prompt` | Prompt to select elements to recolor | None |false| true |None|None|
| `prompt` | Prompt describing the recoloring operation | None |false| true |None|None|

## Inputs
| Type | Description | Default | Required | Resolved |
|---|---|:---:|:---:|:---:|
| `blob` | Input image blob to recolor | None | true | false |
| `imageName` | Input image name | None | false | false |

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `blob` | Recolored image blob | false |

## Examples

### Search and recolor in an image
```yaml
- step: STABILITY_AI_SEARCH_AND_RECOLOR
  select_prompt: "select all trees"
  prompt: "make them autumn color"
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
