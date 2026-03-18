---
sidebar_position: 1
---

# HUB_CONTENT_EXTRACTION

Extract textual content from a URL using the ProActions-Hub extraction tool. The URL can be provided via the default text input or the `in`/`url` configuration.

## At a glance
- **Category** LLM
- **Version:** 1.0.2
- **Applications:** all
- **Scope:** all
- **Default Service:** HUB

## Inputs
| Type | Description | Default | Required | Resolved |
|---|---|:---:|:---:|:---:|
| `text` | URL to extract content from. If omitted, the step will try to read the URL from the default text input. | None | true | false |

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `object` | The parsed extraction result object returned by the Hub extraction API. | false |

## Examples

### Extract content from a URL
```yaml
- step: SET
  text: "https://example.com/article"
- step: HUB_CONTENT_EXTRACTION
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
