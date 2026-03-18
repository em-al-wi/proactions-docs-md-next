---
sidebar_position: 1
---

# PLATFORM

Routes flow execution to platform-specific branches (`swing`, `prime`, `standalone`) with optional `default` fallback.

## At a glance
- **Category** Control
- **Version:** 1.2.0
- **Applications:** all
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `swing` | Steps executed when running in Swing. | None |false| false |None|None|
| `prime` | Steps executed when running in Prime. | None |false| false |None|None|
| `standalone` | Steps executed when running in Standalone mode. | None |false| false |None|None|
| `default` | Fallback steps executed when no platform-specific branch matches. | None |false| false |None|None|

### Step-Level Validation Rules

**Require At Least One:**
- At least one of: swing, prime, standalone, default

## Examples

### Platform-specific branch execution
```yaml
 - step: PLATFORM
   swing:
     - step: SHOW_NOTIFICATION
       message: "Running Swing path"
   prime:
     - step: SHOW_NOTIFICATION
       message: "Running Prime path"
   default:
     - step: DEBUG
       id: "platform-fallback"
       omitDebugger: true
 ```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
