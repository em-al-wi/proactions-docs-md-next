---
sidebar_position: 1
---

# HUB_YOUTUBE_UPLOAD

Upload a video to YouTube via the Hub service. Supports video file, optional thumbnail, metadata fields (title, description, privacyStatus, categoryId), tags and additionalMetadata. Progress can be shown when cfg.updateProgress is true.

## At a glance
- **Category** YouTube
- **Version:** 1.0.12
- **Applications:** all
- **Scope:** all
- **Default Service:** HUB

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `title` | Video title | None |false| true |None|None|
| `description` | Video description | None |false| true |None|None|
| `privacyStatus` | Privacy status: public, unlisted, or private | None |false| false |None|None|
| `categoryId` | YouTube category id | None |false| false |None|None|
| `tags` | Array of tags to set on the uploaded video | None |false| false |None|None|
| `additionalMetadata` | Optional object with additional metadata to attach | None |false| false |None|None|
| `updateProgress` | If true, progress will be reported to ProgressBar during upload | None |false| false |None|None|
| `account` | Account identifier for the Hub target (resolved) | None |false| true |None|None|

## Inputs
| Type | Description | Default | Required | Resolved |
|---|---|:---:|:---:|:---:|
| `video` | Primary video file or blob to upload | None | true | false |
| `thumbnail` | Optional thumbnail file/blob | None | false | false |

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `object` | Result object returned by the upload endpoint (video metadata) | false |

## Examples

### Upload a video to YouTube
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

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
