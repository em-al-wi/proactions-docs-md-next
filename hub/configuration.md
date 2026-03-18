---
id: configuration
title: Setup & Configuration
sidebar_label: Setup & Configuration
sidebar_position: 2
---

# Setup & Initial Configuration

This guide covers the initial configuration steps for ProActions Hub after deployment in a Docker or Podman environment.

## Prerequisites

Before configuring ProActions Hub, ensure you have:

- A running Docker or Podman container with ProActions Hub deployed
- Access to the installation directory (typically `/methode/methdig/tools/proactions-hub/`)
- Necessary API keys for the AI services you plan to use
- Network access to required external services (AI providers, EDAPI, etc.)

## Environment Setup

### Directory Structure

Your ProActions Hub installation should have the following structure:

```
/methode/methdig/tools/proactions-hub/
├── config.yaml              # Main configuration file
├── tokens/                  # YouTube OAuth tokens (optional)
├── .env                     # Environment variables (optional)
├── docker-compose.yml       # Docker Compose config (Docker deployments)
└── proactionshub.bash       # Recreate Pod script (podman deployments)
/methode/methdig/cluster/
└── start_stop_proactionshub.bash  # Control script (Podman deployments)
```

### Configuration File Location

The main configuration file is located at:

```
config.yaml
```

This YAML file uses environment variable substitution with the syntax:

```yaml
key: ${ENVIRONMENT_VARIABLE-default_value}
```

### Environment Variables

You can provide environment variables through:

1. **Container environment** (recommended for production)
2. **`.env` file** in the installation directory
3. **Direct substitution** in `config.yaml` (not recommended for sensitive data)

## Minimal Configuration

### Step 1: Configure Basic Hub Settings

The minimal hub configuration:

```yaml
hub:
```

### Step 2: Configure at Least One AI Target

Add at least one AI service target to enable AI-Link functionality:

```yaml
targets:
  - id: openai
    provider: openai
    apiKey: ${OPENAI_API_KEY}
    options:
      model: gpt-4o-mini
      max_completion_tokens: 12000
```

Set the environment variable:

```bash
OPENAI_API_KEY=sk-your-api-key-here
```

### Step 3: Enable Authentication (Strongly Recommended)

Configure EDAPI authentication:

```yaml
auth:
  enabled: true
  type: edapi
  edapi_url: ${EDAPI_URL}
  session:
    store: memory
    cookie:
      secret: ${PROACTIONS_SESSION_SECRET}
```

Set environment variables:

```bash
EDAPI_URL=https://your-edapi-instance/edapi
PROACTIONS_SESSION_SECRET=your-random-secret-here
```

:::warning Production Security
Always use a strong, randomly-generated session secret in production. Never commit secrets to version control.
:::

### Step 4: Verify Configuration

After making configuration changes:

**For Podman:**

```bash
cd /methode/methdig/tools/proactions-hub/
bash proactionshub.bash
start_stop_proactionshub.bash start
```

**For Docker:**

```bash
cd /methode/methdig/tools/proactions-hub/
docker compose down
docker compose up -d
```

Check if the service is running:

```bash
curl http://localhost:$PROACTIONS_HUB_PORT/health
```

Expected response:

```json
{"status":"ok"}
```

## Volume Mounts

ProActions Hub requires the following volumes to be mounted:

### Configuration Volume (Required)

Mount your configuration file into the container:

```
./config.yaml:/usr/src/app/dist/config.yaml:ro
```

:::tip Read-Only Mount
The configuration file should be mounted read-only (`:ro`) for security.
:::

### Tokens Volume (Required for YouTube)

If using YouTube integration, mount a persistent volume for tokens:

```
./tokens:/usr/src/app/tokens:rw
```

This directory stores OAuth tokens and must be writable by the container.

### Logs Volume (Optional)

For persistent log storage:

```
./logs:/usr/src/app/logs:rw
```

## Network Configuration

### Port Exposure

ProActions Hub uses one port:

- **Main API Port** (default: $PROACTIONS_HUB_PORT) - HTTP/HTTPS API endpoints

Only expose the main API port to your network:

**Podman:**

```bash
-p $PROACTIONS_HUB_PORT:3000
```

**Docker Compose:**

```yaml
ports:
  - "${PROACTIONS_HUB_PORT}:3000
```

### Behind Reverse Proxy / Load Balancer

If deploying behind a reverse proxy or load balancer, configure trust proxy settings:

```yaml
hub:
  security:
    trustProxy: true # or proxy server ip
```

See the [Security Hardening](./operations/security-hardening.md) guide for details.

## Health Checks

ProActions Hub provides health check endpoints for monitoring and load balancer integration.

### Configuration

```yaml
healthCheck:
  enabled: true
  path: /health
  readinessPath: /ready
```

### Endpoints

- **Liveness:** `GET /health` - Returns 200 if the application is running
- **Readiness:** `GET /ready` - Returns 200 if AI-Link service is healthy

### Load Balancer Integration

Configure your load balancer to use:

- **Health check path:** `/health`
- **Healthy status code:** 200
- **Check interval:** 30 seconds (recommended)

## Configuration Validation

ProActions Hub validates configuration on startup using Joi schemas.

### Validation Modes

By default, schema validation is enabled. You can skip validation for development:

```yaml
validation:
  skipSchemaValidation: false  # Skip structure/type validation
```

Environment variables:

```bash
PROACTIONS_SKIP_SCHEMA_VALIDATION=false
```

:::warning Production Warning
Never skip validation in production environments. Validation catches configuration errors early.
:::

## Common Configuration Tasks

### Setting Request Timeouts

Configure HTTP client timeout for AI requests:

```yaml
hub:
  http_client:
    timeout: 120000  # 2 minutes in milliseconds
    maxRedirects: 5
```

### Enabling HTTPS/TLS

Configure TLS certificates:

```yaml
hub:
  tls:
    enabled: true
    cert: ${TLS_CERT_PATH}
    key: ${TLS_KEY_PATH}
    ca: ${TLS_CA_PATH}
```

Mount certificate files into the container and set environment variables.

### Enabling API Documentation

Enable Swagger UI for API testing:

```yaml
swagger:
  enabled: true
  path: docs
```

Access at: `http://localhost:${PROACTIONS_HUB_PORT}/docs`

:::tip Development Only
Disable Swagger in production environments
:::

## Next Steps

Now that you have a basic configuration running:

1. [Configure AI-Link targets](./config/ai-link.md) for your AI providers
2. [Set up logging](./config/logging.md) for monitoring and compliance
3. [Apply security hardening](./operations/security-hardening.md) for production
4. [Review operations guide](./operations/operations-guide.md) for maintenance procedures

## Troubleshooting

### Container Won't Start

Check logs for configuration errors:

**Podman:**

```bash
podman logs -f proactionshub
```

**Docker:**

```bash
docker compose logs -f
```

### Configuration Validation Errors

Look for validation error messages in logs:

```
Configuration validation failed: "hub.port" must be a number
```

Fix the configuration error and restart the service.

### Health Check Fails

If `/health` returns non-200 status:

1. Check that both main port and AI-Link port are available
2. Review logs for AI-Link startup errors
3. Verify network connectivity to AI providers

### Environment Variables Not Substituted

Ensure environment variables are:

- Set before container start
- Properly formatted in config.yaml: `${VAR-default}`
- Not containing special characters that need escaping
