---
sidebar_position: 1
---

# STABILITY_AI_SEARCH_AND_REPLACE

Search and replace regions or objects within an image using StabilityAI. Provide a prompt and a search_prompt to locate and replace visual elements.

## At a glance
- **Category** LLM
- **Aliases** SERVICE_STABILITY_AI_SEARCH_AND_REPLACE
- **Version:** 1.0.4
- **Applications:** all
- **Scope:** all
- **Default Service:** STABILITY_AI

## When to Use

Use **STABILITY_AI_SEARCH_AND_REPLACE** when a described object or region in an image must be located and replaced.

## Data Flow

Reads `blob`, optional `imageName`, and resolved `prompt` and `search_prompt`. Writes the processed image to `blob` or its configured output name.

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `prompt` | Prompt describing replacement target or desired result | None |false| true |None|None|
| `search_prompt` | Prompt describing what to search for in the image | None |false| true |None|None|

## Inputs
| Type | Description | Default | Required | Resolved |
|---|---|:---:|:---:|:---:|
| `blob` | Input image blob to process | None | true | false |
| `imageName` | Input image name | None | false | false |

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `blob` | Processed image blob | false |

## Examples

### Search and replace in an image
```yaml
- step: STABILITY_AI_SEARCH_AND_REPLACE
  search_prompt: "replace red car"
  prompt: "a blue car instead"
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
