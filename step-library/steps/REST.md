---
sidebar_position: 1
---

# REST

Generic REST step. Builds a Request from the given configuration (url, method, headers, parameters, body or formData) and executes it using the host HTTP helper. Results are written to cfg.outputs when provided or to the default text output.

## At a glance
- **Category** Service
- **Aliases** SERVICE_REST
- **Version:** 1.0.0
- **Applications:** all
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `url` | The URL to call. Supports template expressions and variable resolution. | None |true| true |Length:1-∞|None|
| `method` | HTTP method to use (GET, POST, PUT, DELETE, ...). Default: GET. | GET |false| false |None|None|
| `headers` | Optional headers object. Values will be resolved against the flow context. | None |false| false |None|None|
| `parameters` | Optional query parameters object. Values will be resolved against the flow context. | None |false| false |None|None|
| `body` | Optional request body. May be a string or object. Objects are JSON-stringified by the step. | None |false| true |None|Conflicts: formData|
| `formData` | Optional formData object. Supports File/Blob entries and arrays. Values are resolved against the flow context. | None |false| false |None|Conflicts: body|
| `requestOptions` | Optional low-level RequestInit overrides passed to the fetch helper. | None |false| false |None|None|

### Step-Level Validation Rules

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `text` | Default textual response when no outputs are configured | false |
| `json` | Parsed JSON response (if requested via outputs) | true |
| `blob` | Binary blob response (if requested via outputs) | true |
| `bytes` | Raw bytes as Uint8Array (if requested via outputs) | true |
| `arrayBuffer` | ArrayBuffer response (if requested via outputs) | true |
| `object` | Full Response object (if requested via outputs) | true |

## Examples

### Simple GET request
```yaml
- step: REST
  url: "https://api.example.com/items"
  method: "GET"
  outputs:
    - type: json
      name: items
```

### POST request with JSON
```yaml
- step: REST
  url: "https://api.example.com/items"
  method: "POST"
  parameters:
    query: "{{ flowContext.somevar }}"
  headers:
    Authorization: Client-ID XXXXX-XXXXXXXXXXXXXXXXXXXXX
  body:
    myData:
      supportsAlsoVars: {{ flowContext.myVar }}
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
