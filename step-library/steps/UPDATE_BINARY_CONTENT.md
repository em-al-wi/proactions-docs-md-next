---
sidebar_position: 1
---

# UPDATE_BINARY_CONTENT

Update binary content of an existing object in EDAPI using the provided blob. Returns updated object metadata.

## At a glance
- **Category** EDAPI
- **Aliases** CTX_UPDATE_BINARY_CONTENT
- **Version:** 1.0.5
- **Applications:** all
- **Scope:** all

## Inputs
| Type | Description | Default | Required | Resolved |
|---|---|:---:|:---:|:---:|
| `blob` | Binary blob to upload for the object | None | true | false |
| `text` | Id of the object to update | None | true | false |

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `object` | Updated object metadata | false |

## Examples

### Update binary content of an object
```yaml
- step: STABILITY_AI_UPSCALE
  prompt: "{flowContext.imageContent}"
- step: UPDATE_BINARY_CONTENT
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
