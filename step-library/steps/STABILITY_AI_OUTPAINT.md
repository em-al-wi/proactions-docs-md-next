---
sidebar_position: 1
---

# STABILITY_AI_OUTPAINT

Outpaint an image region using StabilityAI. Supports coordinates (left,right,up,down) to define the outpainting area.

## At a glance
- **Category** LLM
- **Aliases** SERVICE_STABILITY_AI_OUTPAINT
- **Version:** 1.0.4
- **Applications:** all
- **Scope:** all
- **Default Service:** STABILITY_AI

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `left` | Pixels to the left (default 0) | None |false| false |None|None|
| `right` | Pixels to the right (default 0) | None |false| false |None|None|
| `up` | Pixels to the top (default 0) | None |false| false |None|None|
| `down` | Pixels to the bottom (default 0) | None |false| false |None|None|

## Inputs
| Type | Description | Default | Required | Resolved |
|---|---|:---:|:---:|:---:|
| `blob` | Input image blob to outpaint | None | true | false |
| `imageName` | Input image name | None | false | false |

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `blob` | Outpainted image blob | false |

## Examples

### Outpaint an image
```yaml
- step: STABILITY_AI_OUTPAINT
  left: 10
  right: 10
  up: 20
  down: 20
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
