---
sidebar_position: 1
---

# STABILITY_AI_UPSCALE

Upscale an image using StabilityAI upscale endpoint. Uploads a provided image blob and returns the upscaled image blob.

## At a glance
- **Category** LLM
- **Aliases** SERVICE_STABILITY_AI_UPSCALE
- **Version:** 1.0.4
- **Applications:** all
- **Scope:** all
- **Default Service:** STABILITY_AI

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `prompt` | Optional prompt describing desired changes to the image | None |false| true |None|None|
| `output_format` | Desired output format (e.g. jpeg, png) | None |false| false |None|None|

## Inputs
| Type | Description | Default | Required | Resolved |
|---|---|:---:|:---:|:---:|
| `blob` | Input image blob to upscale | None | true | false |
| `imageName` | Input image name | None | false | false |

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `blob` | Upscaled image blob | false |

## Examples

### Upscale an image
```yaml
- step: STABILITY_AI_UPSCALE
  output_format: "jpeg"
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
