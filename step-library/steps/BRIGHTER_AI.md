---
sidebar_position: 1
---

# BRIGHTER_AI

Upload an image to BrighterAI, poll for processing completion and download the anonymized image. The resulting anonymized image blob is written to the configured output.

## At a glance
- **Category** Service
- **Aliases** SERVICE_BRIGHTER_AI
- **Version:** 1.0.4
- **Applications:** all
- **Scope:** all
- **Default Service:** BRIGHTER_AI

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `serviceName` | Service operation name (e.g. blur, dnat, mask). If not provided, blur is used. | None |false| false |None|None|
| `params` | Optional url parameters object passed to the BrighterAI upload endpoint. | None |false| false |None|None|

## Inputs
| Type | Description | Default | Required | Resolved |
|---|---|:---:|:---:|:---:|
| `blob` | Input image blob to be processed (can come from previous steps or a flow variable). | None | true | false |
| `imageName` | Optional image name to use when uploading (defaults to "image.jpg"). | None | false | false |

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `blob` | The anonymized image blob returned by the BrighterAI service. | false |

## Examples

### Simple anonymize image
```yaml
- step: BRIGHTER_AI
  serviceName: "blur"
  params:
    face: "true"
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
