# Text-to-Speech Audio Generation

Generate audio versions of articles using AI text-to-speech services.

## Overview

This example demonstrates how to:

- Extract text content while excluding specific elements (`excludeTags`)
- Create dynamic filenames based on article metadata
- Generate audio files using AI speech synthesis (HUB_SPEECH)
- Upload binary files to Méthode

## Requirements

- ProActions Hub configured for content extraction
- OpenAI TTS API access (or compatible service)
- Write permissions to Méthode folders

## Use Case

Modern digital publications often need audio versions of articles for:

- Accessibility (screen reader alternatives)
- Multi-platform distribution (podcasts, smart speakers)
- Mobile consumption (listen while driving)

This workflow automates the creation of high-quality audio narrations.

## Configuration

```yaml
# /SysConfig/ProActions/sections/tts.ai-kit.section.yaml
section: Speech / TTS
keywords: 'Voice'

actions:
  - title: 'Text to Speech (OpenAI)'
    flow:
      # Extract text content, excluding image captions and credits
      - step: SET
        text: '{{ client.getTextContent({ excludeTags: ["web-image-caption", "caption", "credit"] }) | safe }}'
        targetName: "{{ client.getMetadata().metadata.general.title | replace(r/[\\/,\\?]/g, '') | truncate(50) }}.mp3"

      # Generate speech audio
      - step: HUB_SPEECH

      # Upload to Méthode
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

      # Notify user of success
      - step: SHOW_NOTIFICATION
        notificationType: success
        message: "Successfully created audio version of your object."
        title: "Text to Speech"
```

### Main Configuration

```yaml
# pro-actions.ai-kit.yaml
AI_KIT:
  SERVICES: !include /SysConfig/ProActions/services.ai-kit.services.yaml
  SECTIONS:
    - !include /SysConfig/ProActions/sections/tts.ai-kit.section.yaml
```

## How It Works

### Step 1: Extract and Prepare Content

The SET step prepares both the text content and the target filename:

```yaml
- step: SET
  text: '{{ client.getTextContent({ excludeTags: ["web-image-caption", "caption", "credit"] }) | safe }}'
  targetName: "{{ client.getMetadata().metadata.general.title | replace(r/[\\/,\\?]/g, '') | truncate(50) }}.mp3"
```

**Key features:**

**Excluding tags:**

```javascript
client.getTextContent({
  excludeTags: ["web-image-caption", "caption", "credit"]
})
```

This retrieves text content but excludes specified XML elements. Useful for removing:

- Image captions
- Photo credits
- Pull quotes
- Sidebars
- Any non-narrative content

**Dynamic filename:**

```yaml
targetName: "{{ client.getMetadata().metadata.general.title | replace(r/[\\/,\\?]/g, '') | truncate(50) }}.mp3"
```

Creates a safe filename:

- Gets article title from metadata
- Removes invalid filename characters (`/`, `\`, `?`)
- Truncates to 50 characters
- Adds `.mp3` extension

### Step 2: Generate Audio

The HUB_SPEECH step sends text to the speech synthesis service:

```yaml
- step: HUB_SPEECH
```

This step:

- Uses the `text` variable from the previous SET step
- Calls the configured TTS service (e.g., OpenAI TTS)
- Returns binary audio data
- Stores the result in `flowContext.data` by default

### Step 3: Upload to Méthode

The UPLOAD step saves the audio file:

```yaml
- step: UPLOAD
  options:
    name: "{{ flowContext.targetName | safe }}"
    workFolder: "{{ client.getDocumentWorkfolder() }}"
    folderPath: "/Production/Product/Audio"
    type: "Audio"
    options:
      createMode: 'AUTO_RENAME'
```

**Upload options:**

- `name` - Filename for the uploaded file
- `workFolder` - Base Méthode work folder
- `folderPath` - Relative path within work folder
- `type` - Méthode object type
- `createMode: 'AUTO_RENAME'` - Automatically rename if file exists

## Customization

### Configure Voice and Speed

Customize the speech synthesis parameters:

```yaml
- step: HUB_SPEECH
  voice: "alloy"  # OpenAI TTS voices: alloy, echo, fable, onyx, nova, shimmer
  speed: 1.0      # Speed from 0.25 to 4.0
```

### Exclude Different Elements

Customize which elements to exclude:

```yaml
# Exclude more elements
text: '{{ client.getTextContent({
  excludeTags: ["web-image-caption", "caption", "credit", "sidebar", "pullquote", "infobox"]
}) | safe }}'

# Include everything
text: '{{ client.getTextContent() | safe }}'

# Only specific sections
text: '{{ client.getTextContent({
  includeTags: ["text", "headline", "summary"]
}) | safe }}'
```

## Best Practices

1. **Exclude Non-Narrative Content**: Always exclude captions, credits, and other elements that don't belong in narration
2. **Safe Filenames**: Use `replace()` to remove invalid characters and `truncate()` to limit length
3. **User Feedback**: Show notifications on success and provide clear error messages
4. **Auto-Rename**: Use `createMode: 'AUTO_RENAME'` to avoid overwrite conflicts

## Troubleshooting

**Audio quality is poor:**

- Check the TTS service configuration
- Try different voices
- Adjust speed parameter
- Ensure text is clean (no XML tags, special characters)

**Upload fails:**

- Verify folder path exists in Méthode
- Check write permissions
- Ensure `workFolder` is correct
- Check filename for invalid characters

**Wrong content in audio:**

- Verify `excludeTags` list is correct
- Check document structure with `client.getTextContent()` in console
- Test with smaller text selections first

**File already exists error:**

- Use `createMode: 'AUTO_RENAME'` to automatically rename
- Or increment filename manually
- Or use timestamp in filename

## Related Examples

- [Content Import and Rewrite](./content-import.md) - Import and transform content
- [Progress Bar](./progress-bar.md) - Show progress for long operations
