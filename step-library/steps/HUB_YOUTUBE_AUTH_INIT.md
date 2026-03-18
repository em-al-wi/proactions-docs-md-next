---
sidebar_position: 1
---

# HUB_YOUTUBE_AUTH_INIT

Initiate OAuth authorization flow for YouTube (requires ProActions-Hub). Opens the auth URL when available unless cfg.omitOpenAuth is true.

## At a glance
- **Category** YouTube
- **Version:** 1.0.12
- **Applications:** all
- **Scope:** all
- **Default Service:** HUB

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `omitOpenAuth` | If true, do not automatically open the authorization URL in a new window; return the URL in the response instead. | None |false| false |None|None|
| `account` | Optional account identifier used by the Hub API. | None |false| true |None|None|

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `object` | Result object returned by the auth/init call (contains authUrl etc.) | true |

## Examples

### Init YouTube auth
```yaml
- step: HUB_YOUTUBE_AUTH_INIT
  # omitOpenAuth: true
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
