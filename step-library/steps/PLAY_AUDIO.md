---
sidebar_position: 1
---

# PLAY_AUDIO

Plays audio in a lightweight floating player. Audio source can be provided as a data URL, http(s) URL, or raw base64 via `cfg.in` or via the default text input. The step mounts a closed Shadow DOM audio player and resolves when the player is closed by the user.

## At a glance
- **Category** UI
- **Version:** 1.1.0
- **Applications:** all
- **Scope:** all

## Config Options
| Name | Description | Default | Required | Resolved | Constraints | Conditional Rules |
|---|---|:---:|:---:|:---:|---|---|
| `in` | Optional input path or variable name containing the audio source (data URL, http(s) URL or raw base64). If omitted, the default text input is used. | None |false| true |None|None|
| `autoplay` | Attempt autoplay on load. Note: browsers may block autoplay; in that case the player will show a hint and wait for user interaction. | true |false| false |None|None|
| `showProgress` | Show progress / seek bar in the player. | true |false| false |None|None|
| `showVolume` | Show volume controls in the player. | true |false| false |None|None|
| `showTimeDisplay` | Show current time and duration labels. | true |false| false |None|None|
| `theme` | Theme of the player: 'dark' \| 'light' or any valid CSS color string. | dark |false| false |None|None|
| `position` | Player anchoring position: 'top-left' \| 'top-right' \| 'bottom-left' \| 'bottom-right' \| 'top-center' \| 'bottom-center' \| 'center'. | center |false| false |None|None|
| `loop` | If true, loop playback. | false |false| false |None|None|
| `title` | Optional title shown in the player title bar. | None |false| false |None|None|
| `mimeType` | Optional MIME type hint used when the provided input is raw base64 without a data URL header (e.g. "audio/mpeg"). | None |false| false |None|None|

## Inputs
| Type | Description | Default | Required | Resolved |
|---|---|:---:|:---:|:---:|
| `text` | Audio source as data URL, http(s) URL, or raw base64. If omitted, reads from `in` config or default text input. | None | false | false |

## Examples

### Play base64 audio provided via SET
```yaml
- step: SET
  text: "data:audio/mpeg;base64,..."

- step: PLAY_AUDIO
  in: "{{flowContext.text}}"
  autoplay: true
```

### Play audio from an HTTP URL
```yaml
- step: PLAY_AUDIO
  in: "https://example.com/audio.mp3"
  autoplay: false
```

## See Also

**General Resources:**

- [Step Library Overview](../overview.md)
- [Configuration Basics](../../guides/configuration/basics.md)
- [Examples](../../guides/examples/headline-suggestions.md)
