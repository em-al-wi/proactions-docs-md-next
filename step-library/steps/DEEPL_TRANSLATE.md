---
sidebar_position: 1
---

# DEEPL_TRANSLATE

Translate text using DeepL. Supports many optional DeepL parameters and returns the translated text as the default text output.

## At a glance
- **Category** Service
- **Aliases** SERVICE_DEEPL_TRANSLATE
- **Version:** 1.0.0
- **Applications:** all
- **Scope:** all
- **Default Service:** DEEPL

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `instruction` | Text to translate (can include templates). If omitted, the default text input is used. | None |true| true |None|None|
| `target_lang` | Target language code for the translation (e.g. EN, DE, FR). | None |true| true |None|None|
| `replaceXmlLang` | Optional: replace xml:lang attributes in the translated output with this language code. | None |false| true |None|None|
| `tag_handling` | How to handle tags; default is "xml". | None |false| true |None|None|
| `source_lang` | Language of the text to be translated. If this parameter is omitted, the API will attempt to detect the language of the text and translate it. | None |false| true |None|None|
| `context` | Additional context that can influence a translation but is not translated itself. | None |false| true |None|None|
| `split_sentences` | Sets whether the translation engine should first split the input into sentences. | None |false| true |None|None|
| `preserve_formatting` | Sets whether the translation engine should respect the original formatting, even if it would usually | None |false| true |None|None|
| `formality` | Sets whether the translated text should lean towards formal or informal language. | None |false| true |None|None|
| `glossary_id` | Specify the glossary to use for the translation. | None |false| true |None|None|
| `outline_detection` | Disable the automatic detection of XML | None |false| true |None|None|
| `non_splitting_tags` | Comma-separated list of XML tags which never split sentences. | None |false| false |None|None|
| `splitting_tags` | Comma-separated list of XML tags which always cause splits. | None |false| true |None|None|
| `ignore_tags` | Comma-separated list of XML tags that indicate text not to be translated. | None |false| true |None|None|
| `model_type` | Specifies which DeepL model should be used for translation. | None |false| true |None|None|
| `show_billed_characters` | When true, the response will include the billed_characters parameter. | None |false| true |None|None|

## Outputs
| Type | Description | Optional |
|---|---|:---:|
| `text` | Translated text | false |

## Examples

### Translate text using DeepL
```yaml
- step: DEEPL_TRANSLATE
  instruction: "Hello, world!"
  target_lang: "DE"
  # additional parameters as per DeepL API - https://developers.deepl.com/docs/api-reference/translate#request-body-descriptions
  split_sentences: "0"
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
