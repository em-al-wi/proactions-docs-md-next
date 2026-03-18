---
sidebar_position: 1
---

# UPLOAD

Upload content to EDAPI by providing a blob or specifying parameters (filename, type). Returns created object metadata.

## At a glance
- **Category** EDAPI
- **Version:** 1.0.4
- **Applications:** all
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `options` | Obtions data for the UPLOAD | None |false| false |None|None|
| `filename` | Filename to use for the uploaded object | None |false| false |None|None|
| `basetype` | BaseType - Used to identify upload location when using basefolder configuration. | None |false| false |None|None|
| `type` | Object type to create (e.g. File, Image) | None |false| false |None|None|
| `createMode` | Creation mode (e.g. AUTO_RENAME) | None |false| false |None|None|

## Inputs
| Type | Description | Default | Required | Resolved |
|---|---|:---:|:---:|:---:|
| `blob` | Binary blob to upload (preferred) | None | false | false |

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `object` | The created object metadata | false |

## Examples

### Upload with custom options
```yaml
  - step: SET
    targetName: "{{ client.getDocumentName() }}.podcast.mp3"
  - step: UPLOAD
    options:
      name: "{{ flowContext.targetName | safe }}"
      workFolder: "{{ client.getDocumentWorkfolder() }}"
      folderPath: "/Production/Product/Audio"
      type: "Audio"
      attributes: ""
      systemAttributes: "<props></props>"
      application: 'Swing ProActions'
      options:
        showPath: true
        showSystemAttributes: true
        showAttributes: true
        createMode: 'AUTO_RENAME'
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
