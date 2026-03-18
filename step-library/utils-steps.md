# Utils Steps


Steps in the Utils category.


## Available Steps


### [BASE64_TO_BLOB](./steps/BASE64_TO_BLOB.md)

Converts a base64-encoded string to a Blob and stores it in the flow context as a blob output.




**Version:** 1.1.0



**Example:**
```yaml

- step: BASE64_TO_BLOB
  in: "base64Image"
  contentType: "image/png"

```


---


### [MARKDOWN_TO_HTML](./steps/MARKDOWN_TO_HTML.md)

Converts markdown text to HTML and sets it as text output.




**Version:** 1.0.10



**Example:**
```yaml

- step: MARKDOWN_TO_HTML
  text: "# Hello\\nThis is **markdown**"

```


---


### [PARSE_JSON](./steps/PARSE_JSON.md)

Parses JSON content from the text input and stores the resulting object in the flow context.


**Aliases:** PROCESS_PARSE_JSON



**Version:** 1.0.0



**Example:**
```yaml

- step: PARSE_JSON
  in: "rawJson"

```


---


### [SANITIZE](./steps/SANITIZE.md)

Sanitizes text input. Can strip markdown code blocks, normalize Unicode characters (quotes, dashes, spaces), and validate/repair XML content before forwarding the result to the flow context.




**Version:** 1.0.12



**Example:**
```yaml

- step: SANITIZE
  stripMarkdown: true

```


---


### [SCRIPTING](./steps/SCRIPTING.md)

Executes inline JavaScript (`script`) or a JS-style template (`template`). Use with caution — only in trusted flows.




**Version:** 1.0.4



**Example:**
```yaml

- step: SCRIPTING
  script: "flowContext.count = (flowContext.count ?? 0) + 1; return flowContext;"

```


---


### [TO_LIST](./steps/TO_LIST.md)

Convert a textual AI response into a list of strings. Detects JSON arrays, ordered/unordered lists, comma-separated values, or falls back to a single-item array.


**Aliases:** PROCESS_TO_LIST



**Version:** 1.0.0



**Example:**
```yaml

- step: SET
  text: "1. First item\\n2. Second item\\n3. Third item"

- step: TO_LIST

```


---



## Related Documentation

- [Step Library Overview](./overview.md) - Browse all available steps
- [Getting Started](../getting-started/your-first-flow.md) - Learn how to create workflows
