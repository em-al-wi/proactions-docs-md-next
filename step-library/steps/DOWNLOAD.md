---
sidebar_position: 1
---

# DOWNLOAD

Downloads a file constructed from text or an existing Blob. The filename is required.

## At a glance
- **Category** Browser
- **Aliases** DOWNLOAD_TEXT
- **Version:** 1.0.6
- **Applications:** all
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `filename` | Name of the file to download (required). | None |false| false |None|None|
| `contentType` | The content-type to set for the download of the file. | text/plain |false| false |None|None|

## Inputs
| Type | Description | Default | Required | Resolved |
|---|---|:---:|:---:|:---:|
| `filename` | The filename to be used - use cfg.filename instead | None | false | false |

## Examples

### Download a text file
```yaml
- step: SET
  text: "Some text to download"
- step: DOWNLOAD
  filename: "hello.txt"
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
