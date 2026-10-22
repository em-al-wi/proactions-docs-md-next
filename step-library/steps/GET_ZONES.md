---
sidebar_position: 1
---

# GET_ZONES

Retrieves zones and their linked documents from a DWP or Report.

## At a glance
- **Category** Editor
- **Version:** 1.0.0
- **Applications:** all
- **Scope:** dwp, report

## When to Use

Use **GET_ZONES** to retrieve zones and optional linked documents from the active DWP or report for downstream processing.

## Data Flow

Reads no flow context values. Writes the retrieved zones to `list` (or its configured output name) and optionally to a configured `object` output.

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `includeLinks` | Whether to include linked documents in each zone. Defaults to true. | true |false| false |None|None|

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `list` | Array of zone objects, each containing zone info and linked documents. | false |
| `object` | The zones data as an object (same as list). | true |

## Examples

### Get zones with links
```yaml
 - step: GET_ZONES
 ```

### Get zones without links
```yaml
 - step: GET_ZONES
   includeLinks: false
 ```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
