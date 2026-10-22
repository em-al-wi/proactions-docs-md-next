---
sidebar_position: 1
---

# UPLOAD_IMAGE

Upload an image to EDAPI by providing an image object (url or blob). Returns the created object metadata as output.

## At a glance
- **Category** EDAPI
- **Aliases** CTX_UPLOAD_IMAGE
- **Version:** 1.0.0
- **Applications:** all
- **Scope:** all

## When to Use

Use **UPLOAD_IMAGE** to create an EDAPI image object from an image URL or image input.

## Data Flow

Reads `image` or its configured input name. Writes created object metadata to `object` or its configured output name.

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `objectType` | Type of object to create (default: Image). | None |false| false |None|None|
| `createMode` | Creation mode, e.g. AUTO_RENAME. | None |false| false |None|None|

## Inputs
| Type | Description | Default | Required | Resolved |
|---|---|:---:|:---:|:---:|
| `image` | Image input object or blob to upload (can come from file picker or previous step). | None | true | false |

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `object` | The created object metadata returned by EDAPI. | false |

## Examples

### Upload image by URL
```yaml
- step: UPLOAD_IMAGE
  objectType: "Image"
  createMode: "AUTO_RENAME"
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
