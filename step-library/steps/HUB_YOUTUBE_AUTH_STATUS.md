---
sidebar_position: 1
---

# HUB_YOUTUBE_AUTH_STATUS

Check the status of a previously initiated YouTube OAuth authorization flow.

## At a glance
- **Category** YouTube
- **Version:** 1.0.12
- **Applications:** all
- **Scope:** all
- **Default Service:** HUB

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `account` | Optional account identifier used by the Hub API. | None |false| true |None|None|

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `object` | Status object returned by the auth/status endpoint | false |

## Examples

### Check YouTube auth status
```yaml
- step: HUB_YOUTUBE_AUTH_STATUS
  account: "main"
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
