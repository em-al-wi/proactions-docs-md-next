---
sidebar_position: 1
---

# DOCUMENT_PANEL_COMMAND

Executes an array of commands in the document preview panel (e.g. CREATE_PDF, CREATE_TOC)

## At a glance
- **Category** Browser

## When to Use

Use **DOCUMENT_PANEL_COMMAND** to execute preview-panel commands such as generating a table of contents or PDF.

## Data Flow

Reads flow context values referenced by resolved `commands`. Does not write flow context.

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `commands` | Array of strings representing commands to execute (e.g. ["CREATE_TOC", "CREATE_PDF"]) | None |true| false |None|None|

## Examples

### Generate PDF and ToC
```yaml
- step: DOCUMENT_PANEL_COMMAND
  commands:
    - CREATE_TOC
    - CREATE_PDF
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
