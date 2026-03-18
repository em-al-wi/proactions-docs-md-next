---
sidebar_position: 1
---

# END

Ends the execution of a flow based on the given condition, with optional notification.

## At a glance
- **Category** Control
- **Aliases** END_IF
- **Version:** 1.0.12
- **Applications:** all
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `condition` | The condition to evaluate. Flow will be terminated gracefully when condition evaluates to true. | None |false| true |None|None|
| `notification` | Optional notification to show when ending the flow. | None |false| false |None|None|

## Examples

### End flow if condition met
```yaml
 - step: END_IF
   condition: "{{errorOccurred}}"
 ```

### End flow with notification
```yaml
 - step: END_IF
   condition: "{{noDataAvailable}}"
   notification:
     type: "warning"
     message: "No data available for processing"
     title: "Flow Terminated"
 ```

### End flow unconditionally with success notification
```yaml
 - step: END
   notification:
     type: "success"
     message: "Processing completed successfully"
     title: "Done"
 ```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
