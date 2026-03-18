---
sidebar_position: 1
---

# AZURE_OPENAI_IMAGE_GENERATION

ImageGeneration using Azure OpenAI.

:::info Documentation Inheritance
Documentation partly inherited from `OPENAI_IMAGE_GENERATION`.
:::

## At a glance
- **Category** LLM
- **Aliases** SERVICE_AZURE_OPENAI_IMAGE_GENERATION
- **Version:** 1.0.0
- **Applications:** all
- **Scope:** all
- **Default Service:** AZURE_OPENAI_IMAGE_GENERATION

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `model` | Image model to use (gpt-image-1, dall-e-3, dall-e-2) | dall-e-3 |false| false |None|None|
| `instruction` | The user prompt to send to the LLM (supports templates) | None |false| true |Length:1-∞|None|
| `size` | Desired image size (model-dependent) | None |false| false |None|None|
| `quality` | Image quality setting | None |false| false |None|None|
| `n` | Number of images to generate (1-4; DALL·E3 supports 1) | 1 |false| false |Range:1-4|None|
| `style` | Image style for DALL-E 3 (natural, vivid) | None |false| false |None|Conflicts: output_format|
| `response_format` | DALL·E response format: 'url' or 'b64_json' (not supported for gpt-image-1) | None |false| false |None|Conflicts: output_format, background, seed|
| `output_format` | Output format for gpt-image-1 only (png\|jpeg\|webp) | None |false| false |None|Conflicts: response_format, style|
| `background` | Background setting for gpt-image-1 only (transparent) | None |false| false |None|Conflicts: response_format|
| `seed` | Random seed for reproducible results (gpt-image-1 only) | None |false| false |None|Conflicts: response_format|
| `user` | User identifier for attribution and policy monitoring | None |false| false |None|None|
| `timeoutMs` | Request timeout in milliseconds | 60000 |false| false |Range:1000-900000|None|
| `maxRetries` | Maximum number of retries for recoverable errors | 2 |false| false |Range:-∞-20|None|
| `outputKey` | FlowContext key where images will be stored (default: imageList) | imageList |false| false |Pattern: `^[a-zA-Z_][a-zA-Z0-9_]*$`<br/>Length:1-∞|None|

### Step-Level Validation Rules

## Inputs
| Type | Description | Default | Required | Resolved |
|---|---|:---:|:---:|:---:|
| `text` | Prompt text input. If the step reads text input, use a preceding SET step to provide it (e.g. - step: SET text: "..." ). | None | false | false |

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `imageList` | Array of generated image items (each item contains url or b64_json) stored under the outputKey or default key | false |
| `object` | Optional metadata about the generation (model, size, attemptCount) | true |

## Examples

### Generate two images from a prompt
```yaml
- step: SET
  text: "A futuristic skyline at sunset, cinematic lighting"
- step: AZURE_OPENAI_IMAGE_GENERATION
  n: 2
  size: 1024x1024
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
