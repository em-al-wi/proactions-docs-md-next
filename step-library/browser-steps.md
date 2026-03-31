# Browser Steps


Steps in the Browser category.


## Available Steps


### [DIGITAL_PREVIEW](./steps/DIGITAL_PREVIEW.md)

Triggers the digital preview functionality






**Example:**
```yaml

- step: DIGITAL_PREVIEW

```


---


### [DOCUMENT_PANEL_COMMAND](./steps/DOCUMENT_PANEL_COMMAND.md)

Executes an array of commands in the document preview panel (e.g. CREATE_PDF, CREATE_TOC)






**Example:**
```yaml

- step: DOCUMENT_PANEL_COMMAND
  commands:
    - CREATE_TOC
    - CREATE_PDF

```


---


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


### [INSERT_TEMPLATE_SECTION](./steps/INSERT_TEMPLATE_SECTION.md)

Creates a new section in the document using the specified template path, relative to the currently elected item






**Example:**
```yaml

- step: INSERT_TEMPLATE_SECTION
  templatePath: '/Content/Templates/My Awesome Template'
  insertAfter: true

```


---


### [PRINT_PREVIEW](./steps/PRINT_PREVIEW.md)

Triggers the print design preview functionality






**Example:**
```yaml

- step: PRINT_PREVIEW

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
