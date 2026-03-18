# LLM Steps


Steps in the LLM category.


## Available Steps


### [AZURE_OPENAI_COMPLETION](./steps/AZURE_OPENAI_COMPLETION.md)

Chat/completion step that calls OpenAI (or Hub/Azure variants). Supports messages, images, audio, attachments, tools (functions), structured outputs (json_schema/json_object/list), reasoning and audio output. Many step-level configuration options are supported via cfg.*.


**Aliases:** SERVICE_AZURE_OPENAI_COMPLETION



**Version:** 1.0.0



**Example:**
```yaml

- step: AZURE_OPENAI_COMPLETION
  behavior: "You are a helpful assistant."
  instruction: "Write a short summary of the benefits of unit testing. Answer in JSON format."
  options:
    temperature: 0.2
  response_format: "json_object"

```


---


### [AZURE_OPENAI_IMAGE_GENERATION](./steps/AZURE_OPENAI_IMAGE_GENERATION.md)

ImageGeneration using Azure OpenAI.


**Aliases:** SERVICE_AZURE_OPENAI_IMAGE_GENERATION



**Version:** 1.0.0



**Example:**
```yaml

- step: SET
  text: "A futuristic skyline at sunset, cinematic lighting"
- step: AZURE_OPENAI_IMAGE_GENERATION
  n: 2
  size: 1024x1024

```


---


### [AZURE_OPENAI_SPEECH](./steps/AZURE_OPENAI_SPEECH.md)

Azure OpenAI speech step alias.


**Aliases:** SERVICE_AZURE_OPENAI_SPEECH



**Version:** 1.0.6



**Example:**
```yaml

- step: AZURE_OPENAI_SPEECH
  model: "tts-1"
  voice: "alloy"
  response_format: "mp3"

```


---


### [AZURE_OPENAI_TRANSCRIPTION](./steps/AZURE_OPENAI_TRANSCRIPTION.md)

Azure OpenAI transcription alias (delegates to the OpenAI transcription implementation).


**Aliases:** SERVICE_AZURE_OPENAI_TRANSCRIPTION



**Version:** 1.0.0



**Example:**
```yaml

- step: AZURE_OPENAI_TRANSCRIPTION
  model: "whisper-1"

```


---


### [HUB_COMPLETION](./steps/HUB_COMPLETION.md)

Chat/completion step that calls OpenAI (or Hub/Azure variants). Supports messages, images, audio, attachments, tools (functions), structured outputs (json_schema/json_object/list), reasoning and audio output. Many step-level configuration options are supported via cfg.*.




**Version:** 1.0.0



**Example:**
```yaml

- step: HUB_COMPLETION
  behavior: "You are a helpful assistant."
  instruction: "Write a short summary of the benefits of unit testing. Answer in JSON format."
  options:
    temperature: 0.2
  response_format: "json_object"

```


---


### [HUB_CONTENT_EXTRACTION](./steps/HUB_CONTENT_EXTRACTION.md)

Extract textual content from a URL using the ProActions-Hub extraction tool. The URL can be provided via the default text input or the `in`/`url` configuration.




**Version:** 1.0.2



**Example:**
```yaml

- step: SET
  text: "https://example.com/article"
- step: HUB_CONTENT_EXTRACTION

```


---


### [HUB_IMAGE_GENERATION](./steps/HUB_IMAGE_GENERATION.md)

ImageGeneration using ProActions Hub.




**Version:** 1.0.0



**Example:**
```yaml

- step: SET
  text: "A futuristic skyline at sunset, cinematic lighting"
- step: HUB_IMAGE_GENERATION
  n: 2
  size: 1024x1024

```


---


### [HUB_SPEECH](./steps/HUB_SPEECH.md)

ProActionsHub-compatible speech generation step. Inherits behavior from OPENAI_SPEECH.




**Version:** 1.0.6



**Example:**
```yaml

- step: HUB_SPEECH
  model: "tts-1"
  voice: "alloy"
  response_format: "mp3"

```


---


### [HUB_TRANSCRIPTION](./steps/HUB_TRANSCRIPTION.md)

Hub-compatible transcription step (delegates to hub service). Writes transcription text and optional segments list.


**Aliases:** SERVICE_OPENAI_TRANSCRIPTION



**Version:** 1.0.0



**Example:**
```yaml

- step: HUB_TRANSCRIPTION
  model: "whisper-1"

```


---


### [OPENAI_ASSISTANT](./steps/OPENAI_ASSISTANT.md)

Create or reuse an OpenAI assistant. If cfg.assistantId is provided the assistant will be retrieved; otherwise a new assistant is created. The created or retrieved assistant is written to the flow context.




**Version:** 1.0.9



**Example:**
```yaml

- step: OPENAI_ASSISTANT
  assistantId: asst_Ja0fm7Rova3QJGbKrAKrt7HS

```


---


### [OPENAI_COMPLETION](./steps/OPENAI_COMPLETION.md)

Chat/completion step that calls OpenAI (or Hub/Azure variants). Supports messages, images, audio, attachments, tools (functions), structured outputs (json_schema/json_object/list), reasoning and audio output. Many step-level configuration options are supported via cfg.*.


**Aliases:** SERVICE_OPENAI_COMPLETION



**Version:** 1.0.0



**Example:**
```yaml

- step: OPENAI_COMPLETION
  behavior: "You are a helpful assistant."
  instruction: "Write a short summary of the benefits of unit testing. Answer in JSON format."
  options:
    temperature: 0.2
  response_format: "json_object"

```


---


### [OPENAI_DELETE_THREAD](./steps/OPENAI_DELETE_THREAD.md)

Delete a thread using its id. Provide the thread object or id via inputs.




**Version:** 1.0.9



**Example:**
```yaml

- step: OPENAI_DELETE_THREAD
  thread: "{{threadObject}}"

```


---


### [OPENAI_IMAGE_GENERATION](./steps/OPENAI_IMAGE_GENERATION.md)

Generate images using OpenAI image models (DALL·E and GPT-Image-1). Supports model, size, quality, count (n), response_format and other parameters. The generated images are stored in the configured output key or default `imageList`.


**Aliases:** SERVICE_OPENAI_IMAGE_GENERATION



**Version:** 1.0.0



**Example:**
```yaml

- step: SET
  text: "A futuristic skyline at sunset, cinematic lighting"
- step: OPENAI_IMAGE_GENERATION
  n: 2
  size: 1024x1024

```


---


### [OPENAI_SPEECH](./steps/OPENAI_SPEECH.md)

Generate speech audio using the OpenAI Speech API (or corresponding Hub/Azure variants). Reads text from the default text input and returns an audio blob.


**Aliases:** SERVICE_OPENAI_SPEECH



**Version:** 1.0.6



**Example:**
```yaml

- step: OPENAI_SPEECH
  model: "tts-1"
  voice: "alloy"
  response_format: "mp3"

```


---


### [OPENAI_THREAD](./steps/OPENAI_THREAD.md)

Create or reuse an OpenAI thread. If cfg.reuse and cfg.storeIn are provided the thread id may be reused. The created thread is written to the flow context.




**Version:** 1.0.9



**Example:**
```yaml

- step: OPENAI_THREAD
  reuse: true
  storeIn: "session"

```


---


### [OPENAI_THREAD_FILES](./steps/OPENAI_THREAD_FILES.md)

Create a vector store for thread file tools and optionally upload files. Writes created store to flow context and optional uploaded file list to an optional output.




**Version:** 1.0.9



**Example:**
```yaml

- step: OPENAI_THREAD_FILES
  replaceStores: true
  expires_after: 1
  name: "my-store"

```


---


### [OPENAI_THREAD_MESSAGE](./steps/OPENAI_THREAD_MESSAGE.md)

Send a message to a thread/assistant. The assistant and thread inputs are required. The message can be provided via cfg.instruction or via default text input. The assistant response text is stored in the default text output.




**Version:** 1.0.9



**Example:**
```yaml

- step: OPENAI_ASSISTANT # load assistant
  assistantId: asst_Ja0fm2Rova2QJGbKrAKrtXXX
- step: OPENAI_THREAD # load an existing thread
  reuse: true
  storeIn: session
- step: OPENAI_THREAD_MESSAGE
  instruction: |
    Create a list of the key takeaways from the documents provided.

```


---


### [OPENAI_TRANSCRIPTION](./steps/OPENAI_TRANSCRIPTION.md)

Transcribe audio using OpenAI (or Azure/Hub variants). Reads an audio file/blob from inputs and writes the transcription text and optional segment list.


**Aliases:** SERVICE_OPENAI_TRANSCRIPTION



**Version:** 1.0.0



**Example:**
```yaml

- step: OPENAI_TRANSCRIPTION
  model: "whisper-1"

```


---


### [STABILITY_AI_OUTPAINT](./steps/STABILITY_AI_OUTPAINT.md)

Outpaint an image region using StabilityAI. Supports coordinates (left,right,up,down) to define the outpainting area.


**Aliases:** SERVICE_STABILITY_AI_OUTPAINT



**Version:** 1.0.4



**Example:**
```yaml

- step: STABILITY_AI_OUTPAINT
  left: 10
  right: 10
  up: 20
  down: 20

```


---


### [STABILITY_AI_SEARCH_AND_RECOLOR](./steps/STABILITY_AI_SEARCH_AND_RECOLOR.md)

Search and recolor elements in an image using StabilityAI. Provide select_prompt and prompt to find and recolor elements.


**Aliases:** SERVICE_STABILITY_AI_SEARCH_AND_RECOLOR



**Version:** 1.0.4



**Example:**
```yaml

- step: STABILITY_AI_SEARCH_AND_RECOLOR
  select_prompt: "select all trees"
  prompt: "make them autumn color"

```


---


### [STABILITY_AI_SEARCH_AND_REPLACE](./steps/STABILITY_AI_SEARCH_AND_REPLACE.md)

Search and replace regions or objects within an image using StabilityAI. Provide a prompt and a search_prompt to locate and replace visual elements.


**Aliases:** SERVICE_STABILITY_AI_SEARCH_AND_REPLACE



**Version:** 1.0.4



**Example:**
```yaml

- step: STABILITY_AI_SEARCH_AND_REPLACE
  search_prompt: "replace red car"
  prompt: "a blue car instead"

```


---


### [STABILITY_AI_UPSCALE](./steps/STABILITY_AI_UPSCALE.md)

Upscale an image using StabilityAI upscale endpoint. Uploads a provided image blob and returns the upscaled image blob.


**Aliases:** SERVICE_STABILITY_AI_UPSCALE



**Version:** 1.0.4



**Example:**
```yaml

- step: STABILITY_AI_UPSCALE
  output_format: "jpeg"

```


---



## Related Documentation

- [Step Library Overview](./overview.md) - Browse all available steps
- [Getting Started](../getting-started/your-first-flow.md) - Learn how to create workflows
