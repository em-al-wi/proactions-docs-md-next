---
id: proxy
title: Proxy Configuration
sidebar_label: Proxy
sidebar_position: 4
---

# Proxy Configuration

The Proxy module enables HTTP request forwarding to configured target services. This is useful for routing requests through ProActions Hub to external APIs while maintaining centralized authentication and logging.

## Quick Start

### Minimal Proxy Configuration

A basic proxy requires an ID and target URL:

```yaml
proxies:
  - id: deepl
    url: https://api.deepl.com/v2/translate
    options:
      secure: true
      changeOrigin: true
      headers:
        Authorization: ${DEEPL_AUTHORIZATION}
```

Set the environment variable:
```bash
DEEPL_AUTHORIZATION="DeepL-Auth-Key your-api-key"
```

### Using the Proxy

Forward requests using the proxy endpoint:

```bash
curl -X POST http://localhost:${PROACTIONS_HUB_PORT}/proxy/forward/deepl \
  -H "Content-Type: application/json" \
  -d '{
    "text": ["Hello, world!"],
    "target_lang": "ES"
  }'
```

The request path after `/proxy/forward/{id}` is appended to the target URL.

## Proxy Configuration Structure

Each proxy in the `proxies` array has this structure:

```yaml
proxies:
  - id: string                    # Required: Unique identifier
    url: string                   # Required: Target URL
    remove_source_headers: array  # Optional: Headers to strip
    options: object               # Optional: http-proxy options
```

### Required Fields

| Field | Description |
|-------|-------------|
| `id` | Unique identifier used in the proxy path (`/proxy/forward/{id}`) |
| `url` | Target URL to forward requests to |

### Optional Fields

| Field | Description | Default |
|-------|-------------|---------|
| `remove_source_headers` | Array of header names to remove from forwarded requests | `[]` |
| `options` | Configuration options passed to [http-proxy](https://www.npmjs.com/package/http-proxy#options) | `{}` |

## Proxy Options

The `options` field accepts all [http-proxy configuration options](https://www.npmjs.com/package/http-proxy#options). Common options:

### Security Options

```yaml
options:
  secure: true              # Verify SSL certificates
  changeOrigin: true        # Change origin header to target URL
```

### Custom Headers

Add headers to forwarded requests:

```yaml
options:
  headers:
    Authorization: ${API_KEY}
    X-Custom-Header: value
    User-Agent: ProActions-Hub/1.0
```

### Timeout Settings

```yaml
options:
  timeout: 30000           # Socket timeout (ms)
  proxyTimeout: 30000      # Proxy timeout (ms)
```

### WebSocket Support

```yaml
options:
  ws: true                 # Enable WebSocket proxying
```

## Common Proxy Configurations

### DeepL Translation API

```yaml
proxies:
  - id: deepl
    url: https://api.deepl.com/v2/translate
    options:
      secure: true
      changeOrigin: true
      headers:
        Authorization: ${DEEPL_AUTHORIZATION}
```

Usage:
```bash
curl -X POST http://localhost:${PROACTIONS_HUB_PORT}/proxy/forward/deepl \
  -H "Content-Type: application/json" \
  -d '{
    "text": ["Hello, world!"],
    "target_lang": "ES"
  }'
```

### Azure OpenAI (Direct Proxy)

```yaml
proxies:
  - id: azure-openai
    url: https://your-resource.openai.azure.com
    options:
      secure: true
      changeOrigin: true
      headers:
        api-key: ${AZURE_OPENAI_API_KEY}
```

Usage:
```bash
curl -X POST http://localhost:${PROACTIONS_HUB_PORT}/proxy/forward/azure-openai/openai/deployments/gpt-4o/chat/completions?api-version=2024-02-15-preview \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```

:::tip AI-Link vs Proxy
For AI services, prefer using [AI-Link configuration](ai-link.md) which provides normalization and additional features. Use proxy for non-AI APIs or when you need direct passthrough.
:::

### OpenAI (Direct Proxy)

```yaml
proxies:
  - id: openai
    url: https://api.openai.com/v1
    options:
      secure: true
      changeOrigin: true
      headers:
        Authorization: ${OPENAI_AUTHORIZATION}
```

Environment variable:
```bash
OPENAI_AUTHORIZATION="Bearer sk-your-api-key"
```

Usage:
```bash
curl -X POST http://localhost:${PROACTIONS_HUB_PORT}/proxy/forward/openai/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o-mini",
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```

### Generic REST API

```yaml
proxies:
  - id: external-api
    url: https://api.example.com
    options:
      secure: true
      changeOrigin: true
      headers:
        X-API-Key: ${EXTERNAL_API_KEY}
        Accept: application/json
```

## Advanced Configuration

### Removing Source Headers

Strip headers from the original request before forwarding:

```yaml
proxies:
  - id: secure-proxy
    url: https://api.example.com
    remove_source_headers:
      - x-forwarded-for
      - x-real-ip
      - authorization
    options:
      headers:
        Authorization: ${TARGET_API_KEY}
```

This prevents client headers from being forwarded to the target.

### Custom SSL/TLS Configuration

Configure SSL certificate validation:

```yaml
proxies:
  - id: custom-tls
    url: https://api.example.com
    options:
      secure: true
      ssl:
        rejectUnauthorized: true
        ca: ${CA_CERT_PATH}
        cert: ${CLIENT_CERT_PATH}
        key: ${CLIENT_KEY_PATH}
```

### Follow Redirects

```yaml
proxies:
  - id: with-redirects
    url: https://api.example.com
    options:
      followRedirects: true
      maxRedirects: 5
```

### Custom Target Resolution

For dynamic target resolution based on request:

```yaml
proxies:
  - id: dynamic-target
    url: https://api.example.com
    options:
      router: true  # Custom routing logic
```

:::info Advanced Routing
For complex routing logic, you may need to extend the proxy controller. Contact your development team for custom implementations.
:::

## Path Handling

### Default Behavior

The proxy appends the request path after the target URL:

**Configuration:**
```yaml
proxies:
  - id: api
    url: https://api.example.com/v2
```

**Request:** `GET /proxy/forward/api/users/123`

**Forwarded to:** `GET https://api.example.com/v2/users/123`

### Root Path Forwarding

To forward to the root of the target:

**Configuration:**
```yaml
proxies:
  - id: api
    url: https://api.example.com
```

**Request:** `GET /proxy/forward/api/v2/users/123`

**Forwarded to:** `GET https://api.example.com/v2/users/123`

## Security Considerations

### API Key Protection

Always use environment variables for API keys:

```yaml
proxies:
  - id: secure
    url: https://api.example.com
    options:
      headers:
        Authorization: ${API_KEY}  # Good
        # Authorization: sk-abc123  # Bad - never hardcode
```

### SSL Certificate Validation

Always enable SSL verification for production:

```yaml
options:
  secure: true  # Good - verify SSL certificates
  # secure: false  # Bad - only for development
```

### Header Filtering

Remove sensitive headers before forwarding:

```yaml
proxies:
  - id: filtered
    url: https://api.example.com
    remove_source_headers:
      - authorization
      - cookie
      - x-api-key
    options:
      headers:
        Authorization: ${TARGET_API_KEY}
```

### Origin Header

Use `changeOrigin` to prevent CORS issues:

```yaml
options:
  changeOrigin: true  # Sets Host header to target URL
```

## Logging

Proxy operations are logged when proxy logging is enabled:

```yaml
logging:
  types:
    proxy:
      enabled: true
      console: true
      file: false
      level: info
```

Log output includes:
- Proxy target ID and URL
- Request method and path
- Response status code
- Request duration
- Error details (if applicable)

See [Logging Configuration](logging.md) for details.

## Testing Proxy Configuration

### Test with cURL

```bash
# Test GET request
curl http://localhost:${PROACTIONS_HUB_PORT}/proxy/forward/deepl

# Test POST request
curl -X POST http://localhost:${PROACTIONS_HUB_PORT}/proxy/forward/deepl \
  -H "Content-Type: application/json" \
  -d '{"text": ["Hello"], "target_lang": "ES"}'
```

### Test with Swagger UI

If Swagger is enabled:

1. Navigate to `http://localhost:${PROACTIONS_HUB_PORT}/docs`
2. Find the `/proxy/forward/{target}/*` endpoint
3. Enter target ID and path
4. Execute test request

### Verify Logging

Enable proxy logging and check logs:

```bash
# Podman
podman logs -f proactionshub | grep proxy

# Docker
docker compose logs -f | grep proxy
```

## Troubleshooting

### Proxy Target Not Found

**Error:** `Proxy target not found: xyz`

**Solution:** Verify the proxy ID in config.yaml matches the URL path.

### SSL Certificate Error

**Error:** `unable to verify the first certificate`

**Solution:**
1. Set `secure: false` for testing (not recommended for production)
2. Configure proper CA certificates
3. Verify the target URL uses valid SSL certificates

### Connection Timeout

**Error:** `socket hang up` or timeout errors

**Solution:** Increase timeout settings:
```yaml
options:
  timeout: 60000
  proxyTimeout: 60000
```

### CORS Errors

**Error:** Cross-origin request blocked

**Solution:** Ensure `changeOrigin: true` is set:
```yaml
options:
  changeOrigin: true
```

### Headers Not Forwarded

**Issue:** Custom headers not appearing at target

**Solution:**
1. Check `remove_source_headers` is not removing them
2. Add headers explicitly in `options.headers`
3. Verify header names are correct (case-sensitive)

### URL Path Issues

**Issue:** Incorrect path at target

**Solution:** Review proxy URL configuration:
- Ensure target URL doesn't have trailing slash if not needed
- Verify request path structure
- Check that path components are correctly joined

## Best Practices

### Configuration

1. **Use environment variables** for all credentials
2. **Enable SSL verification** in production
3. **Set appropriate timeouts** to prevent hanging requests
4. **Use descriptive IDs** for easy identification

### Security

1. **Remove sensitive headers** before forwarding
2. **Add authentication headers** in proxy configuration
3. **Use changeOrigin** to prevent header leakage
4. **Enable logging** for audit trails

### Performance

1. **Set reasonable timeouts** based on target API
2. **Configure connection pooling** if supported
3. **Monitor proxy logs** for performance issues
4. **Use keepAlive** for better performance

### Reliability

1. **Configure error handling** appropriately
2. **Set retry logic** if needed (custom implementation)
3. **Monitor target availability** through health checks
4. **Log all proxy operations** for troubleshooting

## Next Steps

- [Configure YouTube integration](youtube.md) for video uploads
- [Configure Tools](tools.md) for content extraction
- [Configure Logging](logging.md) for monitoring proxy operations
