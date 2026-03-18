---
sidebar_position: 1
---

# INSERT_OBJECT

Insert object references or templates into the current document using a DTX template.

Prime: Supports paths, UUIDs, and LOIDs (auto-formatted). Options insertionType, full, and templateType are ignored (warnings logged).

## At a glance
- **Category** Editor
- **Version:** 1.2.0
- **Applications:** all
- **Scope:** story, report

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `in` | Optional input variable. String values are treated as path, arrays as referenceListIds. | None |false| true |None|None|
| `path` | Single object path to insert (convenience option). | None |false| true |None|None|
| `referenceListIds` | List of object identifiers (paths, loids, or uuids). | None |false| true |None|None|
| `referenceListObjects` | List of full object entries to insert when `full` is true. | None |false| true |None|None|
| `full` | If true, uses `referenceListObjects` as full object payload. | false |false| false |None|None|
| `withPath` | When using IDs, indicates that identifiers are paths. | true |false| false |None|None|
| `insertionType` | Insertion type for Swing (insert, embed, link). Prime ignores this option. | insert |false| false |None|None|
| `templateName` | Template name to use for insertion. | None |false| false |None|None|
| `templateType` | Template type (Swing only). | None |false| false |None|None|
| `forceWrite` | Write content even if the document is considered read-only. | None |false| false |None|None|

## Inputs
| Type | Description | Default | Required | Resolved |
|---|---|:---:|:---:|:---:|
| `list` | Optional list of object IDs/paths used when `referenceListIds` is not provided. | None | false | false |

## Examples

### Insert object
```yaml
- step: INSERT_OBJECT
  path: "/content/picture/chart.png"
```

### Insert object with template
```yaml
- step: INSERT_OBJECT
  path: "/content/picture/chart.png"
  templateName: "Insert Image as Interactive Figure"
```

### Insert multiple objects
```yaml
- step: INSERT_OBJECT
  in: 
    - "33$1.0.30412980"
    - "/Production/Product/Pictures/Hold/image.jpg"
  templateName: "Inline image"
```

### Insert object list by IDs
```yaml
- step: INSERT_OBJECT
  referenceListIds:
    - "/Globe/Images/Foreign/11.obama_family.jpeg"
    - "/Globe/Images/Foreign/1420087334.jpg"
  withPath: true
  insertionType: insert
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
