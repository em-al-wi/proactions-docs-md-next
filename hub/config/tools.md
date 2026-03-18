---
id: tools
title: Tools Configuration
sidebar_label: Tools
sidebar_position: 6
---

# Tools Configuration

The Tools module provides utility functions for content processing and extraction. This guide covers configuration and usage of available tools.

## Quick Start

### Minimal Configuration

Configure the article extractor tool:

```yaml
tools:
  - id: extract
    type: article_extractor
```

No additional configuration or API keys are required for the article extractor.

### Using the Tool

Extract content from a URL:

```bash
curl http://localhost:${PROACTIONS_HUB_PORT}/tools/extract?url=https://example.com/article
```

Response:
```json
{
  "title": "Article Title",
  "description": "Article description",
  "author": "Author Name",
  "published": "2024-01-15T10:30:00.000Z",
  "content": "Full article text...",
  "links": ["https://..."],
  "image": "https://example.com/image.jpg"
}
```

## Tools Configuration Structure

Each tool in the `tools` array has this structure:

```yaml
tools:
  - id: string           # Required: Unique identifier
    type: string         # Required: Tool type
    options: object      # Optional: Tool-specific options
```

### Required Fields

| Field | Description |
|-------|-------------|
| `id` | Unique identifier for the tool (used for logging and references) |
| `type` | Tool type - currently only `article_extractor` is supported |

### Optional Fields

| Field | Description | Default |
|-------|-------------|---------|
| `options` | Tool-specific configuration options | `{}` |

## Available Tools

### Article Extractor

The article extractor uses [@extractus/article-extractor](https://www.npmjs.com/package/@extractus/article-extractor) to extract content from web pages.

#### Configuration

```yaml
tools:
  - id: extract
    type: article_extractor
```

#### Endpoint

```
GET /tools/extract?url={url}
```

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `url` | string | Yes | URL of the web page to extract |

#### Response Fields

The extractor returns the following fields (when available):

| Field | Description |
|-------|-------------|
| `title` | Article title |
| `description` | Meta description or summary |
| `author` | Author name(s) |
| `published` | Publication date (ISO 8601) |
| `content` | Main article text content |
| `links` | Array of URLs found in the article |
| `image` | Featured image URL |
| `url` | Source URL |
| `source` | Source domain |

#### Example Usage

**Basic extraction:**
```bash
curl http://localhost:${PROACTIONS_HUB_PORT}/tools/extract?url=https://blog.example.com/article-title
```

**Response:**
```json
{
  "title": "How to Configure ProActions Hub",
  "description": "A comprehensive guide to configuring ProActions Hub",
  "author": "Jane Developer",
  "published": "2024-01-15T10:30:00.000Z",
  "content": "ProActions Hub is a powerful tool...",
  "links": [
    "https://docs.example.com",
    "https://github.com/example/repo"
  ],
  "image": "https://blog.example.com/images/cover.jpg",
  "url": "https://blog.example.com/article-title",
  "source": "blog.example.com"
}
```

#### Supported Content Types

The article extractor works best with:
- Blog posts and articles
- News articles
- Documentation pages
- Medium articles
- WordPress sites
- Generic HTML pages with article content

#### Limitations

The extractor may have difficulty with:
- JavaScript-heavy single-page applications (SPAs)
- Pages requiring authentication
- Paywalled content
- Dynamically loaded content
- PDFs and non-HTML documents


## Logging

Tool operations are logged when tools logging is enabled:

```yaml
logging:
  types:
    tools:
      enabled: true
      console: true
      file: false
      level: info
```

Log output includes:
- Tool type and ID
- Request URL
- Extraction success/failure
- Processing time
- Error details (if applicable)

See [Logging Configuration](logging.md) for details.

## Error Handling

### Common Errors

#### Invalid URL

**Error:**
```json
{
  "statusCode": 400,
  "message": "Invalid URL parameter"
}
```

**Solution:** Ensure the URL is properly formatted and URL-encoded.

#### Extraction Failed

**Error:**
```json
{
  "statusCode": 500,
  "message": "Failed to extract content from URL"
}
```

**Solutions:**
1. Verify the URL is accessible
2. Check if the page requires authentication
3. Try a different URL with clearer article structure
4. Check network connectivity from the container

#### Timeout

**Error:**
```json
{
  "statusCode": 504,
  "message": "Request timeout"
}
```

**Solution:** The target website may be slow or unresponsive. Consider configuring HTTP client timeout in hub settings.

### Error Response Structure

All errors follow the NestJS standard format:

```json
{
  "statusCode": 400,
  "message": "Error description",
  "error": "Bad Request"
}
```

## Advanced Configuration

### HTTP Client Timeout

Configure timeout for tool HTTP requests in hub settings:

```yaml
hub:
  http_client:
    timeout: 120000  # 2 minutes for slow websites
    maxRedirects: 5
```

### Custom User Agent

Tools use the same HTTP client configuration as the hub. To set a custom user agent, you would need to extend the tools module (contact your development team).

## Testing Tools Configuration

### Test Article Extraction

```bash
# Test with a known article URL
curl http://localhost:${PROACTIONS_HUB_PORT}/tools/extract?url=https://example.com/article

# Test with URL encoding
curl "http://localhost:${PROACTIONS_HUB_PORT}/tools/extract?url=https%3A%2F%2Fexample.com%2Farticle"
```

### Test with Swagger UI

If Swagger is enabled:

1. Navigate to `http://localhost:${PROACTIONS_HUB_PORT}/docs`
2. Find the `/tools/extract` endpoint
3. Enter a URL
4. Execute the request

### Verify Logging

Enable tools logging and check logs:

```bash
# Podman
podman logs -f proactionshub | grep tools

# Docker
docker compose logs -f | grep tools
```

## Best Practices

### Usage

1. **URL encode parameters** - Always URL-encode the URL parameter
2. **Handle extraction failures** - Not all pages can be extracted successfully
3. **Cache results** - Consider caching extraction results to reduce load
4. **Validate URLs** - Validate URLs client-side before sending requests

### Security

1. **Validate target URLs** - Be cautious of SSRF attacks
2. **Rate limiting** - Consider implementing rate limiting for extraction endpoints
3. **Content filtering** - Be aware that extracted content is user-controlled

### Performance

1. **Set appropriate timeouts** - Configure HTTP client timeout for slow websites
2. **Implement caching** - Cache extraction results in your application
3. **Monitor extraction times** - Track slow extractions through logging
4. **Use async processing** - For large batches, process extractions asynchronously

### Reliability

1. **Handle errors gracefully** - Extraction can fail for many reasons
2. **Implement retries** - Retry failed extractions with exponential backoff
3. **Log extraction failures** - Monitor failure rates and patterns
4. **Provide fallbacks** - Have alternative content sources when extraction fails

## Future Tool Extensions

The tools module is designed to be extensible. Future tools might include:

- PDF text extraction
- Image processing
- Audio transcription
- Video processing
- Document conversion

:::info Custom Tools
Contact your development team if you need custom tools added to the module.
:::

## Next Steps

- [Configure Logging](logging.md) to monitor tool operations
- [Review Security Hardening](../operations/security-hardening.md) for production deployments
- [Operations Guide](../operations/operations-guide.md) for monitoring and troubleshooting
