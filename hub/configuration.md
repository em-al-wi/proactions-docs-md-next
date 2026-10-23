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
aiTargets:
  - id: openai
    provider: openai
    apiKey: ${OPENAI_API_KEY}
    options:
      model: gpt-6-luna
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

:::warning[Production Security]
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

:::tip[Read-Only Mount]
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
- **Readiness:** `GET /ready` - Returns 200 when the memory and disk checks, MCP, the Realtime listener (when enabled), and — only when legacy `targets` exist — the legacy AI gateway are healthy

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

:::warning[Production Warning]
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

:::tip[Development Only]
Disable Swagger in production environments
:::

## Next Steps

Now that you have a basic configuration running:

1. [Configure AI-Link targets](./config/ai-link.md) for your AI providers
2. [Set up logging](./config/logging.md) for monitoring and compliance
3. [Apply security hardening](./operations/security-hardening.md) for production
4. [Review operations guide](./operations/operations-guide.md) for maintenance procedures

## Connecting Your Workflows to Hub

Once Hub is running with at least one AI target, you can use it from any ProActions workflow.
Below is the complete path from Hub configuration to a working YAML action.

### 1. Hub server — define a target (`config.yaml`)

```yaml
# config.yaml (on the Hub server)
aiTargets:
  - id: openai          # logical name you choose — used as "target" in YAML
    provider: openai
    apiKey: ${OPENAI_API_KEY}
    options:
      model: gpt-6-luna
      max_completion_tokens: 12000
```

### 2. ProActions YAML — declare the service

Point the `HUB` service entry at the Hub endpoint and specify which target to use:

```yaml
# /SysConfig/pro-actions.ai-kit.yaml
AI_KIT:
  SERVICES:
    HUB:
      endpoint: '{BASE_URL}/swing/proactions'   # {BASE_URL} = current Swing instance
      target: openai                            # must match target.id in config.yaml
      defaultBehavior: |
        You are a professional writing assistant.
        Answer concisely and in the language of the article.

  SECTIONS:
    - !include /SysConfig/ProActions/sections/my-actions.ai-kit.section.yaml
```

### 3. ProActions YAML — use the service in a flow

Any `HUB_COMPLETION`, `HUB_SPEECH`, `HUB_TRANSCRIPTION`, or `HUB_IMAGE_GENERATION` step
automatically routes through the `HUB` service you declared above:

```yaml
# /SysConfig/ProActions/sections/my-actions.ai-kit.section.yaml
section: Content Generation
context: story
actions:
  - title: 'Summarize Article'
    flow:
      # Call the AI service through Hub — no API key in YAML
      - step: HUB_COMPLETION
        instruction: 'Write a two-sentence summary of this article: {textContent}'

      # Display the result and let the user accept or cancel
      - step: SHOW_RESPONSE
        message: '{{ flowContext.text }}'

    onError:
      - step: SHOW_NOTIFICATION
        notificationType: error
        message: 'Hub unavailable: {{ flowContext.error }}'
```

### Hub Voice Agent (Realtime WebSocket)

`HUB_VOICE_AGENT` does not use the HTTP `/v1` endpoints above. It opens a persistent WebSocket to the dedicated `/proactions/realtime` relay and therefore requires a separate service base URL and an OpenAI Realtime-capable target.

```yaml
# config.yaml on the Hub server
aiTargets:
  - id: openai-realtime
    provider: openai
    apiKey: ${OPENAI_API_KEY}
    options:
      model: gpt-realtime-2.1
    realtime:
      enabled: true
```

The `hub.realtime` block (origin policy, safety secret, session and rate limits) must also be enabled, and main authentication must be `auth.type: token` or `edapi`. See [Realtime Relay](./config/realtime.md).

```yaml
# /SysConfig/ProActions/services.ai-kit.services.yaml
HUB_REALTIME:
  # HUB_VOICE_AGENT appends /proactions/realtime.
  endpoint: '{BASE_URL}/swing'
  target: openai-realtime
  model: gpt-realtime-2.1
```

```yaml
- step: HUB_VOICE_AGENT
  service: HUB_REALTIME
  behavior: |
    You are a helpful voice assistant.
    Call end_session only after the user confirms they are done.
```

Before enabling this flow, verify that the deployed Hub version includes the Realtime relay. Requires ProActions Hub 1.3.0 or later. The Hub or its ingress must:

- Expose `WSS /proactions/realtime`
- Accept the EDAPI or Hub token and target metadata sent through `Sec-WebSocket-Protocol`
- Preserve WebSocket upgrade and subprotocol headers through every reverse proxy
- Use trusted TLS when the CMS is served over HTTPS
- Allow long-lived connections for at least the configured voice-session duration
- Permit outbound WSS access to `api.openai.com:443`

The client must run in Swing or Prime with an available EDAPI token. Browser CSP and firewalls must permit WSS access to Hub. A Hub deployment that exposes only the endpoints listed under [AI-Link](./config/ai-link.md#available-endpoints) is not sufficient.

AI-Kit verifies both connection phases automatically before microphone capture: the browser-to-Hub upgrade must select the `realtime` subprotocol, and Hub must return OpenAI's `session.created` event within 15 seconds. Failures identify whether to inspect the relay/proxy/authentication phase or the target/credential/egress phase.

Verify the deployment by starting a Hub voice flow with browser developer tools open. The Network panel must show `/proactions/realtime?model=...` upgrading with status `101` and selecting the `realtime` subprotocol. Hub logs must show the request ID, selected target and model (user identities are never logged), and successful upstream OpenAI connection. Interpret `404` as a missing relay, `401`/`403` as authentication or subprotocol-forwarding failure, and an upstream connection error as target credentials or egress failure.

See [Voice Agents](../guides/configuration/voice-agents.md) for browser requirements, complete flow examples, VAD controls, tools, outputs, and troubleshooting.

### 4. Switching targets at the action level

If you have multiple Hub targets configured (e.g., `openai` and `claude`), you can override the
default at the step level without changing the service declaration:

```yaml
# config.yaml — two targets
aiTargets:
  - id: openai
    provider: openai
    apiKey: ${OPENAI_API_KEY}
    options:
      model: gpt-6.1-sol

  - id: claude
    provider: anthropic
    apiKey: ${ANTHROPIC_API_KEY}
    options:
      model: claude-opus-5-5
```

```yaml
# YAML — per-step target override
- step: HUB_COMPLETION
  instruction: 'Draft a headline: {textContent}'
  target: claude   # overrides the section-level HUB.target for this step only
```

### 5. Using a different service definition at the step level

Both options above (`target:` within a single service, multiple service declarations) share
the same `HUB` service definition. A third option is to declare **multiple service definitions**
in `SERVICES:` and tell a specific step to use one of them via the `service:` property.

This is useful when two service definitions need different `defaultBehavior`, `endpoint`, or
other settings that cannot be changed per-step via `target:` alone:

```yaml
# /SysConfig/pro-actions.ai-kit.yaml
AI_KIT:
  SERVICES:
    HUB:
      endpoint: '{BASE_URL}/swing/proactions'
      target: openai
      defaultBehavior: |
        You are a professional writing assistant.

    HUB_TRANSLATE:
      endpoint: '{BASE_URL}/swing/proactions'
      target: openai
      defaultBehavior: |
        You are a professional translator.
        Preserve formatting and tone.
        Return only the translated text, no explanations.
```

```yaml
# In a flow — pick the service definition per step
- step: HUB_COMPLETION
  instruction: 'Improve this text: {selectedText}'
  # service: not set — uses the default HUB service

- step: HUB_COMPLETION
  instruction: 'Translate to French: {selectedText}'
  service: HUB_TRANSLATE   # uses the HUB_TRANSLATE service definition instead
```

`service:` is supported on **all** service-aware steps, not just `HUB_COMPLETION`. For example,
if you have two DeepL configurations pointing to different API plans, you can switch between them
the same way:

```yaml
SERVICES:
  DEEPL:
    endpoint: /swing/proactions/proxy/forward/deepl
  DEEPL_PRO:
    endpoint: /swing/proactions/proxy/forward/deepl-pro

# In flow:
- step: DEEPL_TRANSLATE
  instruction: '{selectedText}'
  target_lang: FR
  service: DEEPL_PRO   # use the PRO plan service definition for this step
```

For the full list of `HUB_COMPLETION` options, see the
[HUB_COMPLETION step reference](../step-library/steps/HUB_COMPLETION.md).

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

1. `/health` is liveness only: a non-200 means the process itself is unhealthy — check the container logs for startup or configuration errors.
2. For `/ready`, inspect the JSON body for the failing check. The legacy gateway check (`ai_gateway`) only applies while legacy `targets` exist; otherwise it reports `skipped: true`.
3. Verify network connectivity to AI providers.

### Environment Variables Not Substituted

Ensure environment variables are:

- Set before container start
- Properly formatted in config.yaml: `${VAR-default}`
- Not containing special characters that need escaping
