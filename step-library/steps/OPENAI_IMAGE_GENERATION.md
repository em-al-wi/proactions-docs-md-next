---
sidebar_position: 1
---

# OPENAI_IMAGE_GENERATION

Generate images using OpenAI GPT Image models (default gpt-image-2.5-sunburst). Supports model, size, quality, count (n), output_format and background. The generated images are stored in the configured output key or default `imageList`.

## At a glance
- **Category** LLM
- **Aliases** SERVICE_OPENAI_IMAGE_GENERATION
- **Version:** 1.0.0
- **Applications:** all
- **Scope:** all
- **Default Service:** OPENAI_IMAGE_GENERATION

## When to Use

Use **OPENAI_IMAGE_GENERATION** when a prompt from the flow must generate one or more images with an OpenAI image model.

## Data Flow

Reads resolved `instruction` or `text`. Writes generated items to `outputKey` (default `imageList`) and optionally writes generation metadata to a configured `object` output.

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `model` | Image model (default gpt-image-2.5-sunburst). Current OpenAI models: gpt-image-2.5-sunburst, gpt-image-2.5-flare, gpt-image-2, gpt-image-1.5, gpt-image-1-mini | gpt-image-2.5-sunburst |false| false |None|None|
| `instruction` | The user prompt to send to the LLM (supports templates) | None |false| true |Length:1-∞|None|
| `size` | Image size: 1024x1024 (default), 1536x1024 (landscape) or 1024x1536 (portrait) | None |false| false |None|None|
| `quality` | Image quality (default auto). 'xhigh' and 'max' require a GPT Image 2.5 model; other models receive 'high'. | None |false| false |None|None|
| `n` | Number of images to generate (1-4) | 1 |false| false |Range:1-4|None|
| `output_format` | Output file format: png (default), jpeg or webp | None |false| false |None|None|
| `background` | Background: auto, transparent (needs output_format png or webp) or opaque | None |false| false |None|None|
| `user` | User identifier for attribution and policy monitoring | None |false| false |None|None|
| `timeoutMs` | Request timeout in milliseconds | 60000 |false| false |Range:1000-900000|None|
| `maxRetries` | Maximum number of retries for recoverable errors | 2 |false| false |Range:-∞-20|None|
| `outputKey` | FlowContext key where images will be stored (default: imageList) | imageList |false| false |Pattern: `^[a-zA-Z_][a-zA-Z0-9_]*$`<br/>Length:1-∞|None|

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
- step: OPENAI_IMAGE_GENERATION
  n: 2
  size: 1024x1024
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
