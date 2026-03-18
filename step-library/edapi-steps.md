# EDAPI Steps


Steps in the EDAPI category.


## Available Steps


### [EDAPI_OBJECT_CONTENT](./steps/EDAPI_OBJECT_CONTENT.md)

Fetch object content from EDAPI and return it as blob or configured outputs. The content id can be provided via cfg.contentId or via a text input.




**Version:** 1.0.4



**Example:**
```yaml

- step: SELECTED_OBJECT
- step: EDAPI_OBJECT_CONTENT
  # contentId: "{{ flowContext.text }}"
  format: lowres

```


---


### [UPDATE_BINARY_CONTENT](./steps/UPDATE_BINARY_CONTENT.md)

Update binary content of an existing object in EDAPI using the provided blob. Returns updated object metadata.


**Aliases:** CTX_UPDATE_BINARY_CONTENT



**Version:** 1.0.5



**Example:**
```yaml

- step: STABILITY_AI_UPSCALE
  prompt: "{flowContext.imageContent}"
- step: UPDATE_BINARY_CONTENT

```


---


### [UPLOAD](./steps/UPLOAD.md)

Upload content to EDAPI by providing a blob or specifying parameters (filename, type). Returns created object metadata.




**Version:** 1.0.4



**Example:**
```yaml

  - step: SET
    targetName: "{{ client.getDocumentName() }}.podcast.mp3"
  - step: UPLOAD
    options:
      name: "{{ flowContext.targetName | safe }}"
      workFolder: "{{ client.getDocumentWorkfolder() }}"
      folderPath: "/Production/Product/Audio"
      type: "Audio"
      attributes: ""
      systemAttributes: "<props></props>"
      application: 'Swing ProActions'
      options:
        showPath: true
        showSystemAttributes: true
        showAttributes: true
        createMode: 'AUTO_RENAME'

```


---


### [UPLOAD_IMAGE](./steps/UPLOAD_IMAGE.md)

Upload an image to EDAPI by providing an image object (url or blob). Returns the created object metadata as output.


**Aliases:** CTX_UPLOAD_IMAGE



**Version:** 1.0.0



**Example:**
```yaml

- step: UPLOAD_IMAGE
  objectType: "Image"
  createMode: "AUTO_RENAME"

```


---



## Related Documentation

- [Step Library Overview](./overview.md) - Browse all available steps
- [Getting Started](../getting-started/your-first-flow.md) - Learn how to create workflows
