# YouTube Steps


Steps in the YouTube category.


## Available Steps


### [HUB_YOUTUBE_AUTH_INIT](./steps/HUB_YOUTUBE_AUTH_INIT.md)

Initiate OAuth authorization flow for YouTube (requires ProActions-Hub). Opens the auth URL when available unless cfg.omitOpenAuth is true.




**Version:** 1.0.12



**Example:**
```yaml

- step: HUB_YOUTUBE_AUTH_INIT
  # omitOpenAuth: true

```


---


### [HUB_YOUTUBE_AUTH_LOGOUT](./steps/HUB_YOUTUBE_AUTH_LOGOUT.md)

Log out the configured Hub YouTube account. Uses POST to logout and returns the resulting object.




**Version:** 1.0.12



**Example:**
```yaml

- step: HUB_YOUTUBE_AUTH_LOGOUT
  account: "main"

```


---


### [HUB_YOUTUBE_AUTH_STATUS](./steps/HUB_YOUTUBE_AUTH_STATUS.md)

Check the status of a previously initiated YouTube OAuth authorization flow.




**Version:** 1.0.12



**Example:**
```yaml

- step: HUB_YOUTUBE_AUTH_STATUS
  account: "main"

```


---


### [HUB_YOUTUBE_UPLOAD](./steps/HUB_YOUTUBE_UPLOAD.md)

Upload a video to YouTube via the Hub service. Supports video file, optional thumbnail, metadata fields (title, description, privacyStatus, categoryId), tags and additionalMetadata. Progress can be shown when cfg.updateProgress is true.




**Version:** 1.0.12



**Example:**
```yaml

- step: HUB_YOUTUBE_UPLOAD
  account: "main"
  title: "My video"
  description: "An example upload"
  privacyStatus: "unlisted"
  tags:
    - example
    - demo

```


---



## Related Documentation

- [Step Library Overview](./overview.md) - Browse all available steps
- [Getting Started](../getting-started/your-first-flow.md) - Learn how to create workflows
