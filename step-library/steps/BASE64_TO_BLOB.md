---
sidebar_position: 1
---

# BASE64_TO_BLOB

Converts a base64-encoded string to a Blob and stores it in the flow context as a blob output.

## At a glance
- **Category** Utils
- **Version:** 1.1.0
- **Applications:** all
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `in` | Optional path or variable name to read the base64 string from the flowContext. If omitted, the default text input is used. | None |false| true |None|None|
| `contentType` | Optional content type for the resulting Blob (e.g. "image/png"). | None |false| false |None|None|

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `blob` | The resulting Blob created from the base64 input. | false |

## Examples

### Convert base64 to blob
```yaml
- step: BASE64_TO_BLOB
  in: "base64Image"
  contentType: "image/png"
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
