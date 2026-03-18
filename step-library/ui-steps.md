# UI Steps


User interaction steps handle communication with users through forms, prompts, notifications, and menus.


## Available Steps


### [CHANGE_VIEW_SIZE](./steps/CHANGE_VIEW_SIZE.md)

Requests a change of the view port size. Useful to resize the command palette view in Prime 8.




**Version:** 1.0.12



**Example:**
```yaml

- step: CHANGE_VIEW_SIZE
  width: 800
  height: 600

```


---


### [FILE_UPLOAD](./steps/FILE_UPLOAD.md)

Opens a file upload dialog (modal) and stores selected file(s) into the flow context. Supports output mapping via the `outputs` config or default file output.


**Aliases:** USER_FILE_UPLOAD



**Version:** 1.0.0



**Example:**
```yaml

- step: FILE_UPLOAD
  promptText: "Please upload your document"

```


---


### [FORM](./steps/FORM.md)

Displays a configurable form to the user (via FormBuilder). The form definition is provided in `cfg.form`. When submitted, the returned values are merged into the flowContext.




**Version:** 1.0.2



**Example:**
```yaml

- step: FORM
  title: "Your info"
  form:
    firstName:
      type: text
      label: "First name"
      required: true
    lastName:
      type: text
      label: "Last name"
      required: true

```


---


### [HIDE_PROGRESS](./steps/HIDE_PROGRESS.md)

Hide any shown progress bar.




**Version:** 1.0.8



**Example:**
```yaml

- step: HIDE_PROGRESS

```


---


### [IMAGE_PICKER](./steps/IMAGE_PICKER.md)

Shows an image picker modal allowing the user to pick one image from a provided list of image objects (\{ url, previewUrl?, label? \}). The selected image object is stored as an output.


**Aliases:** USER_IMAGE_PICKER



**Version:** 1.0.0



**Example:**
```yaml

- step: USER_IMAGE_PICKER
  promptText: "Choose an image"
  imageList:
    - url: "https://example.com/image1.jpg"
      previewUrl: "https://example.com/thumb1.jpg"
      label: "Image 1"
    - url: "https://example.com/image2.jpg"
      previewUrl: "https://example.com/thumb2.jpg"
      label: "Image 2"

```


---


### [MONITOR_CHOICE](./steps/MONITOR_CHOICE.md)

Display a selection of choices in the Flow Monitor. Pauses workflow until user selects.




**Version:** 1.2.0



**Example:**
```yaml

- step: MONITOR_CHOICE
  message: "Select export format:"
  choices:
    - id: pdf
      label: "📄 PDF Document"
    - id: docx
      label: "📝 Word Document"
    - id: html
      label: "🌐 HTML Page"

- step: EXPORT_DOCUMENT
  format: "{{flowContext.text}}"
  
```


---


### [MONITOR_CONFIRM](./steps/MONITOR_CONFIRM.md)

Display a confirmation dialog in the Flow Monitor. Pauses workflow until user responds.




**Version:** 1.2.0



**Example:**
```yaml

- step: MONITOR_CONFIRM
  message: "Publish this article?"

- step: IF
  condition: "{{flowContext.confirmed}}"
  then:
    - step: PUBLISH_ARTICLE
  
```


---


### [MONITOR_INPUT](./steps/MONITOR_INPUT.md)

Display an inline text input field in the Flow Monitor.




**Version:** 1.2.0



**Example:**
```yaml

- step: MONITOR_INPUT
  message: "Enter filename:"
  placeholder: "document"
  outputs:
    - type: text
      name: filename
  
```


---


### [MONITOR_JSON](./steps/MONITOR_JSON.md)

Display JSON structured data in the Flow Monitor.




**Version:** 1.0.0



**Example:**
```yaml

- step: MONITOR_JSON
  title: "Model response"
  data: "{{flowContext.object}}"
  maxDepth: 4
  
```


---


### [MONITOR_KEYVALUE](./steps/MONITOR_KEYVALUE.md)

Display key-value structured data in the Flow Monitor.




**Version:** 1.0.0



**Example:**
```yaml

- step: MONITOR_KEYVALUE
  title: "Result summary"
  data: "{{flowContext.object}}"
  
```


---


### [MONITOR_METRICS](./steps/MONITOR_METRICS.md)

Display metrics in the Flow Monitor.




**Version:** 1.0.0



**Example:**
```yaml

- step: MONITOR_METRICS
  title: "Evaluation"
  data:
    score: 87
    confidence: 0.92
    tokens: 1420
  
```


---


### [MONITOR_STATUS](./steps/MONITOR_STATUS.md)

Display status messages in the Flow Monitor during workflow execution.




**Version:** 1.2.0



**Example:**
```yaml

- step: MONITOR_STATUS
  message: "Connecting to OpenAI API..."
  type: status

- step: OPENAI_COMPLETION
  # ... OpenAI step config

- step: MONITOR_STATUS
  message: "Response received!"
  type: success
  
```


---


### [MONITOR_TABLE](./steps/MONITOR_TABLE.md)

Display tabular structured data in the Flow Monitor.




**Version:** 1.0.0



**Example:**
```yaml

- step: MONITOR_TABLE
  title: "Summary"
  data: "{{flowContext.list}}"
  maxRows: 20
  
```


---


### [PLAY_AUDIO](./steps/PLAY_AUDIO.md)

Plays audio in a lightweight floating player. Audio source can be provided as a data URL, http(s) URL, or raw base64 via `cfg.in` or via the default text input. The step mounts a closed Shadow DOM audio player and resolves when the player is closed by the user.




**Version:** 1.1.0



**Example:**
```yaml

- step: SET
  text: "data:audio/mpeg;base64,..."

- step: PLAY_AUDIO
  in: "{{flowContext.text}}"
  autoplay: true

```


---


### [PROMPT](./steps/PROMPT.md)

Shows a simple prompt to the user and stores the entered text into the default text output.


**Aliases:** USER_PROMPT



**Version:** 1.0.0



**Example:**
```yaml

- step: PROMPT
  promptText: "Please enter your name"

```


---


### [SHOW_NOTIFICATION](./steps/SHOW_NOTIFICATION.md)

Shows a notification to the user with optional title and resolved message.


**Aliases:** CTX_SHOW_NOTIFICATION



**Version:** 1.0.0



**Example:**
```yaml

 - step: SHOW_NOTIFICATION
   notificationType: "info"
   message: "Processing completed for {{userName}}"
   title: "Done"
 
```


---


### [SHOW_PROGRESS](./steps/SHOW_PROGRESS.md)

Show a progress bar with optional status text. Use `position` to place it and `autoHide` to auto-hide after completion.




**Version:** 1.0.8



**Example:**
```yaml

- step: SHOW_PROGRESS
  status: "Processing..."
  position: "bottom"
  autoHide: true

```


---


### [SHOW_RESPONSE](./steps/SHOW_RESPONSE.md)

Shows a response to the user in a modal. Can run inline steps before showing and supports HTML and Markdown modes.


**Aliases:** CTX_SHOW_RESPONSE



**Version:** 1.0.0



**Example:**
```yaml

 - step: SET
   text: "Text to show"
 - step: SHOW_RESPONSE
 
```


---


### [UPDATE_PROGRESS](./steps/UPDATE_PROGRESS.md)

Update an existing progress bar. Provide percentage (0-100) and optional status text.




**Version:** 1.0.8



**Example:**
```yaml

- step: UPDATE_PROGRESS
  percentage: "{{progressPercent}}"
  status: "Step {{progressStep}}/5"

```


---


### [USER_SELECT](./steps/USER_SELECT.md)

Displays a selection modal allowing the user to pick one item from a list. The list may be provided directly or produced by a pre-processing step (inlineSteps). The selected value is written to the default text output.




**Version:** 1.0.0



**Example:**
```yaml

- step: USER_SELECT
  promptText: "Choose an option"
  # list can be taken from prior steps or from the flowContext

```


---



## Related Documentation

- [Step Library Overview](./overview.md) - Browse all available steps
- [Getting Started](../getting-started/your-first-flow.md) - Learn how to create workflows
