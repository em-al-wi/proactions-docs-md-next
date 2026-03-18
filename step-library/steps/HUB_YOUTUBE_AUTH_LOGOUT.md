---
sidebar_position: 1
---

# HUB_YOUTUBE_AUTH_LOGOUT

Log out the configured Hub YouTube account. Uses POST to logout and returns the resulting object.

## At a glance
- **Category** YouTube
- **Version:** 1.0.12
- **Applications:** all
- **Scope:** all
- **Default Service:** HUB

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `account` | Account identifier to log out (resolved). | None |false| true |None|None|

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `object` | Result from the logout endpoint | false |

## Examples

### Logout YouTube account
```yaml
- step: HUB_YOUTUBE_AUTH_LOGOUT
  account: "main"
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
