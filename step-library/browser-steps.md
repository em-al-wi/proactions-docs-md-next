# Browser Steps


Steps in the Browser category.


## Available Steps


### [DOWNLOAD](./steps/DOWNLOAD.md)

Downloads a file constructed from text or an existing Blob. The filename is required.


**Aliases:** DOWNLOAD_TEXT



**Version:** 1.0.6



**Example:**
```yaml

- step: SET
  text: "Some text to download"
- step: DOWNLOAD
  filename: "hello.txt"

```


---


### [READ_CLIPBOARD](./steps/READ_CLIPBOARD.md)

Reads content from the user clipboard. Prefers HTML content when available, falls back to plain text.




**Version:** 1.0.3



**Example:**
```yaml

- step: READ_CLIPBOARD

```


---


### [WRITE_CLIPBOARD](./steps/WRITE_CLIPBOARD.md)

Writes text to the user clipboard. Reads the content from the configured input (cfg.text) or from the default text input in the flow context.




**Version:** 1.0.3



**Example:**
```yaml

- step: SET
  text: "Some text to copy"

- step: WRITE_CLIPBOARD

```


---



## Related Documentation

- [Step Library Overview](./overview.md) - Browse all available steps
- [Getting Started](../getting-started/your-first-flow.md) - Learn how to create workflows
