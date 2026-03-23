# Content Import and Rewrite

Import content from external URLs and transform it using AI with structured output.

## Overview

This example demonstrates how to:

- Extract content from remote URLs (HUB_CONTENT_EXTRACTION)
- Use the `| safe` filter to preserve XML formatting
- Upload linked images to the Méthode (UPLOAD_IMAGE)
- Use EDAPI REST calls to correlate content
- Transform content using structured JSON output

## Requirements

- ProActions Hub for content extraction
- OpenAI or Google Gemini for structured output
- Write permissions for image uploads

## Use Case

Authors often need to aggregate content from external sources like:

- Police reports and press releases
- Government announcements
- Wire services and news feeds
- Social media posts

This workflow automates content import and transforms it into publication-ready articles.

## Configuration

```yaml
# /SysConfig/ProActions/sections/content-robot.ai-kit.section.yaml
section: ProActions Tools
keywords: ''

actions:
  #
  # Step 1 - Import Content
  #
  - title: 'Content Robot - Import'
    flow:
      # Prompt user for URL
      - step: PROMPT
        promptText: 'URL:'

      # Extract remote content
      - step: HUB_CONTENT_EXTRACTION

      # Write content into active document
      - step: REPLACE_TEXT
        at: XPATH
        xpath: "/doc/story/grouphead/headline/p"
        in: "{{ flowContext.object.title | safe }}"

      - step: REPLACE_TEXT
        at: XPATH
        xpath: "/doc/story/text/p"
        in: "{{ flowContext.object.content | safe }}"

      # Check if there's an image and download it
      - step: IF
        condition: "{{ flowContext.object.image }}"
        then:
          # Create image object as input for UPLOAD_IMAGE step
          - step: SET
            image:
              url: "{{ flowContext.object.image }}"

          # Download the image to the basefolder
          - step: UPLOAD_IMAGE
            objectType: Image

          # Correlate the downloaded image using EDAPI
          - step: REST
            url: "{{ client.getEdapiUrl() }}/v3/object/{{ client.getDocumentId() }}/correlate"
            method: POST
            headers:
              token: "{{ client.getEdapiToken() }}"
            body:
              sourceLinkName: "see_also"
              targetLinkName: "see_also"
              targetSource: "{{ flowContext.object.id }}"

          - step: SHOW_NOTIFICATION
            notificationType: success
            message: "A picture was downloaded and correlated to your story."
            title: "ProActions - Content Robot"

  #
  # Step 2 - Rewrite content
  #
  - title: 'Content Robot - Rephrase and improve article (Reports)'
    keywords: Summary Rephrase
    flow:
      - step: HUB_COMPLETION
        behavior: |
          # Task
          You are a author tasked with transforming a police report into a polished
          newspaper article. Your goal is to maintain the factual integrity while enhancing
          readability and engagement for the general public.

          Follow these steps to create a compelling article:

          Title: Develop a concise and engaging title that reflects the core incident or
          issue, capturing readers' attention.

          Teaser Text: Write a brief teaser (approximately 2-3 sentences) summarizing the
          main event or issue, designed to entice readers to explore further.

          Body Text: Expand the content into a well-structured article. Ensure it includes:
          - An engaging introduction that sets the scene or context.
          - A detailed account of the incident, including key facts and figures.
          - Contextual information, such as background details or related trends.
          - A conclusion that summarizes the main points and, if applicable, suggests
            implications or calls to action.

          # Example
          Police Report: "On October 15th, at 10:00 PM, officers responded to a burglary
          at 123 Main St. The suspect was apprehended on-site and charged with breaking
          and entering."

          Rephrased Article:
          Title: "Suspect Caught in Late-Night Burglary at Local Residence"

          Teaser Text: A quick response by local police led to the arrest of a burglary
          suspect at a residence on Main Street.

          Body Text: On the evening of October 15th, a swift police response resulted in
          the apprehension of a suspect involved in a burglary at 123 Main Street. Officers
          arrived at the scene shortly after receiving an emergency call and managed to catch
          the suspect in the act. The individual has been charged with breaking and entering,
          marking another success in local law enforcement's ongoing efforts to combat property
          crimes. This incident highlights the importance of community vigilance and timely
          reporting in ensuring neighborhood safety.
        instruction: |
          The input text: {{ client.getTextContent() }}
        response_format:
          name: 'article_object'
          schema:
            type: object
            properties:
              title:
                type: string
              teaser:
                type: string
              body:
                type: string
            required:
              - title
              - teaser
              - body

      - step: REPLACE_TEXT
        at: XPATH
        xpath: /doc/story/grouphead/headline/p
        in: "{{ flowContext.object.title | safe }}"

      - step: REPLACE_TEXT
        at: XPATH
        xpath: /doc/story/summary/p
        in: "{{ flowContext.object.teaser | safe}}"

      - step: REPLACE_TEXT
        at: XPATH
        xpath: /doc/story/text/p
        in: "{{ flowContext.object.body | safe }}"
```

### Main Configuration

```yaml
# pro-actions.ai-kit.yaml
AI_KIT:
  SERVICES: !include /SysConfig/ProActions/services.ai-kit.services.yaml
  SECTIONS:
    - !include /SysConfig/ProActions/sections/content-robot.ai-kit.section.yaml
```

## How It Works

### Step 1: Content Extraction

The HUB_CONTENT_EXTRACTION step fetches and parses web content:

```yaml
- step: PROMPT
  promptText: 'URL:'

- step: HUB_CONTENT_EXTRACTION
```

This step:

- Takes a URL from the PROMPT step (stored in `{{flowContext.text}}`)
- Fetches the web page
- Extracts structured content (title, body, images)
- Returns an object with extracted data in `flowContext.object`

**Extracted object structure:**

```javascript
{
  title: "Article title",
  content: "Main content with HTML",
  image: "https://example.com/image.jpg"
}
```

### Step 2: Insert Content with Safe Filter

The `| safe` filter prevents HTML escaping:

```yaml
- step: REPLACE_TEXT
  at: XPATH
  xpath: "/doc/story/text/p"
  in: "{{ flowContext.object.content | safe }}"
```

### Step 3: Conditional Image Download

The workflow checks if an image was extracted:

```yaml
- step: IF
  condition: "{{ flowContext.object.image }}"
  then:
    # Image processing steps
```

### Step 4: Image Upload

The UPLOAD_IMAGE step downloads and saves the image:

```yaml
- step: SET
  image:
    url: "{{ flowContext.object.image }}"

- step: UPLOAD_IMAGE
  objectType: Image
```

This:

- Creates an image object with the URL
- Downloads the image
- Uploads it to Méthode
- Stores the new image ID in `flowContext.object.id`

### Step 5: Correlate Image with EDAPI

The REST step links the image to the article:

```yaml
- step: REST
  url: "{{ client.getEdapiUrl() }}/v3/object/{{ client.getDocumentId() }}/correlate"
  method: POST
  headers:
    token: "{{ client.getEdapiToken() }}"
  body:
    sourceLinkName: "see_also"
    targetLinkName: "see_also"
    targetSource: "{{ flowContext.object.id }}"
```

**Client functions used:**

- `client.getEdapiUrl()` - EDAPI base URL
- `client.getDocumentId()` - Current document ID
- `client.getEdapiToken()` - Authentication token

### Step 6: Structured Content Transformation

The HUB_COMPLETION step with structured output:

```yaml
- step: HUB_COMPLETION
  behavior: |
    Transform a police report into a newspaper article with:
    - Title
    - Teaser
    - Body
  instruction: |
    The input text: {{ client.getTextContent() }}
  response_format:
    name: 'article_object'
    schema:
      type: object
      properties:
        title:
          type: string
        teaser:
          type: string
        body:
          type: string
      required:
        - title
        - teaser
        - body
```

This returns a structured object:

```javascript
{
  title: "Engaging headline",
  teaser: "Brief summary",
  body: "Full article text"
}
```

Accessible as:

- `{{ flowContext.object.title }}`
- `{{ flowContext.object.teaser }}`
- `{{ flowContext.object.body }}`

## Best Practices

1. **Use `| safe` Carefully**: Only use on trusted content to avoid XSS vulnerabilities
2. **Validate URLs**: Check URL format before extraction
3. **Handle Missing Images**: Always check if images exist before processing
4. **Structured Output**: Use JSON schemas for consistent, parseable results
5. **Error Handling**: Wrap external calls in TRY blocks
6. **User Feedback**: Show notifications for success and failure states

## ## Troubleshooting

**Content extraction fails:**

- Verify URL is accessible
- Check if site requires authentication
- Some sites block automated content extraction
- Try the URL in a browser first

**HTML not rendering:**

- Ensure you're using `| safe` filter
- Check if content contains invalid XML
- Use SANITIZE step if content has problematic XML

**Image upload fails:**

- Verify image URL is accessible
- Check Méthode permissions
- Ensure image format is supported
- Check file size limits

**EDAPI correlation fails:**

- Verify EDAPI credentials
- Check that object IDs are valid
- Ensure link names exist in schema
- Check EDAPI logs for detailed errors

## Related Examples

- [Text-to-Speech](./text-to-speech.md) - Generate audio from imported content
- [Image Outpainting](./image-outpainting.md) - Process uploaded images
- [Headline Suggestions](./headline-suggestions.md) - Generate headlines for imported content
