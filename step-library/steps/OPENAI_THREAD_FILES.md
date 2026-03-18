---
sidebar_position: 1
---

# OPENAI_THREAD_FILES

Create a vector store for thread file tools and optionally upload files. Writes created store to flow context and optional uploaded file list to an optional output.

## At a glance
- **Category** LLM
- **Version:** 1.0.9
- **Applications:** all
- **Scope:** all
- **Default Service:** OPENAI_COMPLETION

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `name` | Name of the vector store | None |false| false |None|None|
| `replaceStores` | If true replace existing stores instead of appending | None |false| false |None|None|
| `expires_after` | Expiration configuration for the vector store | None |false| false |None|None|

## Inputs
| Type | Description | Default | Required | Resolved |
|---|---|:---:|:---:|:---:|
| `files` | Optional files to upload to the vector store | None | false | false |
| `thread` | Optional thread object to update with tool resources | None | false | false |

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `store` | Created vector store object | false |
| `files` | Optional uploaded files metadata list | true |

## Examples

### Create vector store and upload files
```yaml
- step: OPENAI_THREAD_FILES
  replaceStores: true
  expires_after: 1
  name: "my-store"
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
