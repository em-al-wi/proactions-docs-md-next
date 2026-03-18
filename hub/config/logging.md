---
id: logging
title: Logging Configuration
sidebar_label: Logging
sidebar_position: 7
---

# Logging Configuration

ProActions Hub provides comprehensive logging capabilities powered by Winston. This guide covers configuration, log types, and best practices for monitoring and compliance.

## Quick Start

### Minimal Configuration

```yaml
logging:
  format: json
  defaultLogLevel: info
  types:
    access:
      enabled: true
      console: true
    error:
      enabled: true
      console: true
```

This enables basic access and error logging to console in JSON format.

## Logging Configuration Structure

```yaml
logging:
  format: string                # Log format: 'json' or 'text'
  text_format: string           # Custom text format template
  defaultLogLevel: string       # Default log level
  logDirectory: string          # Directory for log files
  filePrefix: string            # Prefix for log filenames
  fileRotation: boolean         # Enable file rotation
  maxFileSize: string           # Max size per log file
  maxFiles: string              # Retention period
  sensitiveData: array          # Keywords to filter
  logTrace: boolean             # Include stack traces
  types: object                 # Per-type configurations
```

## Log Formats

### JSON Format (Recommended)

```yaml
logging:
  format: json
```

Example output:
```json
{
  "timestamp": "2024-01-15T10:30:00.123Z",
  "level": "info",
  "message": "AI request completed",
  "module": "aiLink",
  "target": "openai",
  "model": "gpt-4o-mini",
  "promptTokens": 15,
  "completionTokens": 25,
  "totalTokens": 40
}
```

**Advantages:**
- Easy to parse and query
- Works well with log aggregation tools
- Structured data for analytics
- Better for production environments

### Text Format

```yaml
logging:
  format: text
  text_format: "${timestamp} [${level}] [${module}]: ${message}"
```

Example output:
```
2024-01-15T10:30:00.123Z [info] [aiLink]: AI request completed
```

**Advantages:**
- Human-readable
- Good for development
- Easier to read in terminal

## Log Levels

Supported log levels (highest to lowest priority):

| Level | Description | Use Case |
|-------|-------------|----------|
| `error` | Critical errors requiring attention | Production issues, exceptions |
| `warn` | Warning conditions | Deprecated features, recoverable errors |
| `info` | Informational messages | Request tracking, business events |
| `debug` | Detailed debugging information | Development, troubleshooting |
| `verbose` | Very detailed information | Deep debugging |
| `silly` | Most granular logs | Rarely used |

### Default Log Level

```yaml
logging:
  defaultLogLevel: info
```

This sets the level for log types that don't specify their own level.

:::tip Production Recommendation
Use `info` or `warn` in production. Use `debug` only for troubleshooting.
:::

## Log Types

ProActions Hub supports six log types, each configurable independently:

### 1. Access Logs

HTTP request/response logs.

```yaml
logging:
  types:
    access:
      enabled: true
      console: true
      file: true
      level: info
```

**Logged information:**
- HTTP method and URL
- Status code
- Response time
- Client IP address
- User agent
- Request ID
- Username (if authenticated)

**Example:**
```json
{
  "timestamp": "2024-01-15T10:30:00.123Z",
  "level": "info",
  "message": "POST /v1/chat/completions 200 1234ms",
  "method": "POST",
  "url": "/v1/chat/completions",
  "statusCode": 200,
  "responseTime": 1234,
  "ip": "192.168.1.100",
  "userAgent": "ProActions-Client/1.0",
  "requestId": "abc123",
  "username": "john.doe"
}
```

### 2. Error Logs

Application errors and exceptions.

```yaml
logging:
  types:
    error:
      enabled: true
      console: true
      file: true
      level: error
```

**Logged information:**
- Error message
- Stack trace (if `logTrace: true`)
- Request context
- Error code
- Module where error occurred

**Example:**
```json
{
  "timestamp": "2024-01-15T10:30:00.123Z",
  "level": "error",
  "message": "Failed to connect to AI provider",
  "error": "ECONNREFUSED",
  "stack": "Error: ECONNREFUSED...",
  "module": "aiLink",
  "target": "openai"
}
```

### 3. AI-Link Logs

AI operations and token usage tracking.

```yaml
logging:
  types:
    aiLink:
      enabled: true
      console: true
      file: false
      level: info
      logPrompt: false        # Include prompts in logs
      logResponse: false      # Include responses in logs
```

**Logged information:**
- Target and provider
- Model used
- Token usage (prompt, completion, total)
- Request duration
- Optional: Full prompt and response

**Example:**
```json
{
  "timestamp": "2024-01-15T10:30:00.123Z",
  "level": "info",
  "message": "AI request completed",
  "module": "aiLink",
  "target": "openai",
  "provider": "openai",
  "model": "gpt-4o-mini",
  "promptTokens": 150,
  "completionTokens": 250,
  "totalTokens": 400,
  "duration": 1234
}
```

:::warning Prompt/Response Logging
Only enable `logPrompt` and `logResponse` for debugging. These logs can contain sensitive data and consume significant storage.
:::

### 4. Proxy Logs

HTTP proxy operations.

```yaml
logging:
  types:
    proxy:
      enabled: true
      console: true
      file: false
      level: info
```

**Logged information:**
- Proxy target ID and URL
- Request path
- Response status
- Request duration

**Example:**
```json
{
  "timestamp": "2024-01-15T10:30:00.123Z",
  "level": "info",
  "message": "Proxy request completed",
  "module": "proxy",
  "target": "deepl",
  "targetUrl": "https://api.deepl.com/v2/translate",
  "path": "/",
  "statusCode": 200,
  "duration": 456
}
```

### 5. YouTube Logs

YouTube API operations.

```yaml
logging:
  types:
    youtube:
      enabled: true
      console: true
      file: false
      level: info
```

**Logged information:**
- Account ID
- Operation type (auth, upload, etc.)
- Success/failure status
- Video ID (for uploads)

**Example:**
```json
{
  "timestamp": "2024-01-15T10:30:00.123Z",
  "level": "info",
  "message": "Video uploaded successfully",
  "module": "youtube",
  "account": "main",
  "videoId": "dQw4w9WgXcQ",
  "title": "My Video"
}
```

### 6. Tools Logs

Tool operations (content extraction, etc.).

```yaml
logging:
  types:
    tools:
      enabled: true
      console: true
      file: false
      level: info
```

**Logged information:**
- Tool ID and type
- Operation parameters
- Success/failure status
- Processing time

**Example:**
```json
{
  "timestamp": "2024-01-15T10:30:00.123Z",
  "level": "info",
  "message": "Content extraction completed",
  "module": "tools",
  "tool": "extract",
  "url": "https://example.com/article",
  "duration": 789
}
```

## File Logging

### Configuration

```yaml
logging:
  logDirectory: logs
  filePrefix: app
  fileRotation: true
  maxFileSize: 10m
  maxFiles: 14d
```

### File Structure

When file logging is enabled for a log type, files are created with this pattern:

```
logs/
├── app-access-2024-01-15.log
├── app-error-2024-01-15.log
├── app-aiLink-2024-01-15.log
└── ...
```

### File Rotation

With rotation enabled:
- Files rotate daily
- Files rotate when they reach `maxFileSize`
- Old files are deleted after `maxFiles` period
- Files are compressed after rotation (gzip)

**Example maxFileSize values:**
- `10m` - 10 megabytes
- `100m` - 100 megabytes
- `1g` - 1 gigabyte

**Example maxFiles values:**
- `7d` - 7 days
- `14d` - 14 days
- `30d` - 30 days
- `10` - Keep 10 files

## Sensitive Data Filtering

ProActions Hub automatically filters sensitive information from logs.

### Configuration

```yaml
logging:
  sensitiveData:
    - password
    - token
    - apiKey
    - authorization
    - key
    - secret
```

### How It Works

Sensitive values are replaced with `[FILTERED]`:

**Before filtering:**
```json
{
  "apiKey": "sk-abc123xyz",
  "password": "myPassword123"
}
```

**After filtering:**
```json
{
  "apiKey": "[FILTERED]",
  "password": "[FILTERED]"
}
```

### Custom Filters

Add custom patterns:

```yaml
logging:
  sensitiveData:
    - password
    - token
    - apiKey
    - ssn
    - creditCard
    - privateKey
```

## Stack Trace Logging

### Configuration

```yaml
logging:
  logTrace: false
```

When `true`, error logs include full stack traces.

:::warning Production Warning
Stack traces may expose sensitive information about your application's internal structure. Only enable for debugging.
:::

## Complete Configuration Example

### Production Configuration

```yaml
logging:
  format: json
  defaultLogLevel: info
  logDirectory: logs
  filePrefix: proactions
  fileRotation: true
  maxFileSize: 100m
  maxFiles: 30d
  logTrace: false
  sensitiveData:
    - password
    - token
    - apiKey
    - authorization
    - key
    - secret
  types:
    access:
      enabled: true
      console: false
      file: true
      level: info
    error:
      enabled: true
      console: true
      file: true
      level: error
    aiLink:
      enabled: true
      console: false
      file: true
      level: info
      logPrompt: false
      logResponse: false
    proxy:
      enabled: true
      console: false
      file: true
      level: info
    youtube:
      enabled: true
      console: false
      file: true
      level: info
    tools:
      enabled: true
      console: false
      file: true
      level: info
```

### Development Configuration

```yaml
logging:
  format: text
  text_format: "${timestamp} [${level}] [${module}]: ${message}"
  defaultLogLevel: debug
  logTrace: true
  types:
    access:
      enabled: true
      console: true
      file: false
      level: debug
    error:
      enabled: true
      console: true
      file: false
      level: error
    aiLink:
      enabled: true
      console: true
      file: false
      level: debug
      logPrompt: true
      logResponse: true
    proxy:
      enabled: true
      console: true
      file: false
      level: debug
    youtube:
      enabled: true
      console: true
      file: false
      level: debug
    tools:
      enabled: true
      console: true
      file: false
      level: debug
```

## Viewing Logs

### Console Logs

**Podman:**
```bash
podman logs -f proactionshub
```

**Docker:**
```bash
docker compose logs -f
```

### File Logs

```bash
# View all logs
tail -f logs/app-access-*.log

# View specific log type
tail -f logs/app-error-*.log

# View logs with jq (for JSON)
tail -f logs/app-aiLink-*.log | jq '.'
```

### Filter Logs by Level

**Using jq (JSON format):**
```bash
tail -f logs/app-access-*.log | jq 'select(.level == "error")'
```

**Using grep (text format):**
```bash
tail -f logs/app-access-*.log | grep ERROR
```

## Log Aggregation

### ELK Stack

For JSON logs, configure Filebeat to ship logs to Elasticsearch:

```yaml
# filebeat.yml
filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - /path/to/logs/app-*.log
    json.keys_under_root: true
    json.add_error_key: true

output.elasticsearch:
  hosts: ["elasticsearch:9200"]
```

### Splunk

Configure Splunk Universal Forwarder to monitor log directory:

```ini
[monitor:///path/to/logs]
sourcetype = _json
index = proactions
```

### CloudWatch

For AWS deployments, use CloudWatch Logs agent to ship logs.

## Performance Considerations

### High-Traffic Environments

1. **Disable console logging** - Write to files only
   ```yaml
   console: false
   file: true
   ```

2. **Use JSON format** - More efficient to parse
   ```yaml
   format: json
   ```

3. **Increase log level** - Reduce log volume
   ```yaml
   defaultLogLevel: warn
   ```

4. **Disable verbose AI logging** - Don't log prompts/responses
   ```yaml
   logPrompt: false
   logResponse: false
   ```

### Storage Management

1. **Configure rotation** - Prevent disk space issues
   ```yaml
   fileRotation: true
   maxFileSize: 100m
   maxFiles: 14d
   ```

2. **Archive old logs** - Move to cold storage
   ```bash
   find logs/ -name "*.gz" -mtime +30 -exec mv {} /archive/ \;
   ```

## Monitoring Token Usage

Enable AI-Link logging to track token consumption:

```yaml
logging:
  types:
    aiLink:
      enabled: true
      file: true
```

Query logs for token statistics:

```bash
# Total tokens by model (jq)
cat logs/app-aiLink-*.log | jq -r '[.totalTokens] | add'

# Tokens by target
cat logs/app-aiLink-*.log | jq -r 'group_by(.target) | .[] | {target: .[0].target, total: ([.[].totalTokens] | add)}'
```

## Compliance and Auditing

### Audit Trail

Access logs provide complete audit trail:
- Who made requests (username)
- When requests were made (timestamp)
- What endpoints were accessed (URL)
- Request outcomes (status code)

### Data Retention

Configure retention based on compliance requirements:

```yaml
logging:
  maxFiles: 365d  # 1 year retention
```

### Sensitive Data

Ensure sensitive data filtering is enabled:

```yaml
logging:
  sensitiveData:
    - password
    - token
    - apiKey
    - ssn
    - creditCard
```

## Best Practices

### Configuration

1. **Use JSON in production** - Easier to parse and query
2. **Set appropriate log levels** - Balance detail vs. volume
3. **Enable file rotation** - Prevent disk space issues
4. **Configure retention** - Meet compliance requirements

### Security

1. **Filter sensitive data** - Prevent credential leaks
2. **Secure log files** - Appropriate file permissions
3. **Disable stack traces** - Don't expose internal structure
4. **Don't log prompts/responses** - Unless debugging

### Performance

1. **Disable console in production** - Reduce overhead
2. **Use async logging** - Winston handles this automatically
3. **Limit log levels** - Higher levels for production
4. **Monitor log volume** - Adjust levels if needed

### Operations

1. **Monitor disk usage** - Ensure logs don't fill disk
2. **Set up log rotation** - Automatic cleanup
3. **Integrate with aggregation** - Central log management
4. **Create dashboards** - Visualize metrics

## Troubleshooting

### Logs Not Appearing

**Issue:** No logs in console or files

**Solutions:**
1. Check log type is enabled
2. Verify log level allows messages through
3. Check file permissions for log directory
4. Ensure log directory exists and is writable

### Too Many Logs

**Issue:** Excessive log volume

**Solutions:**
1. Increase log level (info → warn → error)
2. Disable verbose AI logging
3. Disable access logs for health checks
4. Filter noisy endpoints

### Log Files Growing Too Large

**Issue:** Disk space consumed by logs

**Solutions:**
1. Enable file rotation
2. Reduce maxFileSize
3. Reduce retention period
4. Compress old logs

### Can't Parse JSON Logs

**Issue:** Invalid JSON in logs

**Solutions:**
1. Verify format is set to `json`
2. Check for multi-line stack traces (set `logTrace: false`)
3. Ensure Winston is properly initialized

## Next Steps

- [Configure Security Hardening](../operations/security-hardening.md) for production
- [Review Operations Guide](../operations/operations-guide.md) for log monitoring procedures
