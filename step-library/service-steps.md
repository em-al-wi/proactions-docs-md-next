# Service Steps


Service integration steps connect to external services like AI providers, translation APIs, and other third-party services.


## Available Steps


### [BRIGHTER_AI](./steps/BRIGHTER_AI.md)

Upload an image to BrighterAI, poll for processing completion and download the anonymized image. The resulting anonymized image blob is written to the configured output.


**Aliases:** SERVICE_BRIGHTER_AI



**Version:** 1.0.4



**Example:**
```yaml

- step: BRIGHTER_AI
  serviceName: "blur"
  params:
    face: "true"

```


---


### [DEEPL_TRANSLATE](./steps/DEEPL_TRANSLATE.md)

Translate text using DeepL. Supports many optional DeepL parameters and returns the translated text as the default text output.


**Aliases:** SERVICE_DEEPL_TRANSLATE



**Version:** 1.0.0



**Example:**
```yaml

- step: DEEPL_TRANSLATE
  instruction: "Hello, world!"
  target_lang: "DE"
  # additional parameters as per DeepL API - https://developers.deepl.com/docs/api-reference/translate#request-body-descriptions
  split_sentences: "0"

```


---


### [DEEPL_WRITE](./steps/DEEPL_WRITE.md)

Rephrase/write text using DeepL Write endpoint. Returns the rephrased text in the default text output.


**Aliases:** SERVICE_DEEPL_WRITE



**Version:** 1.0.9



**Example:**
```yaml

- step: DEEPL_WRITE
  instruction: "This is bad grammar"
  tone: "friendly"

```


---


### [ELEVENLABS_STT](./steps/ELEVENLABS_STT.md)

Transcribe audio using ElevenLabs STT. Provides the transcribed text and optional JSON result.




**Version:** 1.0.12



**Example:**
```yaml

- step: ELEVENLABS_STT
  model_id: "scribe_v1-1"

```


---


### [ELEVENLABS_TTS](./steps/ELEVENLABS_TTS.md)

Generate speech audio from text using ElevenLabs TTS. Text is taken from the default text input or a flow variable via `in`.




**Version:** 1.0.12



**Example:**
```yaml

- step: ELEVENLABS_TTS
  voice_id: "21m00Tcm4TlvDq8ikWAM"
  output_format: "mp3"

```


---


### [HUB_MCP_INVOKE](./steps/HUB_MCP_INVOKE.md)

Invoke a tool on an MCP server via the Hub.




**Version:** 1.2.0



**Example:**
```yaml

- step: HUB_MCP_INVOKE
  server: filesystem
  tool: read_file
  arguments:
    path: /tmp/test.txt

```


---


### [HUB_MCP_TOOLS](./steps/HUB_MCP_TOOLS.md)

List available tools from an MCP server via the Hub.




**Version:** 1.2.0



**Example:**
```yaml

- step: HUB_MCP_TOOLS

```


---


### [REST](./steps/REST.md)

Generic REST step. Builds a Request from the given configuration (url, method, headers, parameters, body or formData) and executes it using the host HTTP helper. Results are written to cfg.outputs when provided or to the default text output.


**Aliases:** SERVICE_REST



**Version:** 1.0.0



**Example:**
```yaml

- step: REST
  url: "https://api.example.com/items"
  method: "GET"
  outputs:
    - type: json
      name: items

```


---



## Related Documentation

- [Step Library Overview](./overview.md) - Browse all available steps
- [Getting Started](../getting-started/your-first-flow.md) - Learn how to create workflows
