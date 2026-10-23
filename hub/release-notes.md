---
id: pah-release-notes
title: ProActions Hub Release Notes
sidebar_label: Hub Release Notes
---

### 1.3.0 - (unreleased)

:::warning[Legacy AI-Link `targets` are deprecated — action required]
Provider entries declared under `targets:` (served by the legacy AI gateway) are deprecated and will be removed in a future release. They keep working in 1.3.x, but every request served by one now logs a deprecation warning naming its id. Migrate them to `aiTargets`; the [migration guide](./config/ai-link-migration.md) covers OpenAI, AWS Bedrock, Anthropic, and Google (both the Gemini API and Vertex AI), plus load balancing, failover and guardrail hooks.
:::

#### New Features

##### New AI Engine (`aiTargets`)

AI-Link moves to a new, built-in AI engine. Targets declared under `aiTargets` run on it directly inside the Hub — no separate gateway process — with native provider authentication, automatic retries, composite routing, guardrails and structured output. Same `x-target` header, same OpenAI wire format — no client change, and an `aiTargets` entry takes priority over a legacy target with the same id.

- 17 native providers: OpenAI, Azure, Anthropic, Google, Vertex, Bedrock, Mistral, Groq, DeepSeek, xAI, Cohere, Together AI, Fireworks, Perplexity, Hugging Face, OpenRouter, and any OpenAI-compatible endpoint
- Provider aliases keep legacy names working (`azure-openai`, `vertex-ai`, `gemini`, `amazon-bedrock`, …)
- Per-target `settings`, `options`, `providerOptions`, `requestTimeout`, `retry`, `debug` and `guardrails`

See [AI-Link Configuration](./config/ai-link.md).

##### Composite Routing (fallback / loadbalance)

One `x-target` id can fan out to an ordered fallback list or a weighted load-balanced set of concrete targets, with per-route weights.

See [Routing, Retries & Timeouts](./config/ai-link-routing.md).

##### Automatic Retries with Backoff

Chat completions and the Responses API retry retryable upstream failures (429, 5xx, timeouts, network errors) by default: 3 total attempts with equal-jitter exponential backoff. Configurable globally under `hub.http_client.retry` and per target under `retry`; `retry: { maxAttempts: 1 }` restores single-shot behaviour.

##### Guardrails

Regex, length, JSON, PII, LLM-judge and moderation-API checks, applied before the request, after the response, or both, with `deny` / `redact` / `log` actions and a separate `onError` policy for check failures.

See [Guardrails](./config/ai-link-guardrails.md).

##### Responses API

`POST /v1/responses` with function tools, `tool_choice`, `instructions`, `text.format` and SSE streaming (aiTargets only).

##### Structured Output

`response_format: json_object` / `json_schema` is now executed for chat completions and the Responses API, so the provider enforces the requested format.

##### Images and Audio on `aiTargets`

Image generation, audio speech, audio transcription and audio translation are served by `aiTargets`. Translation is available on `openai` and `openai-compatible` targets; unsupported provider/modality combinations return a clear `400`.

##### Provider Authentication

- Azure OpenAI through Microsoft Entra ID (`settings.useEntraId`) alongside static keys
- AWS Bedrock through STS role assumption (`settings.roleArn`, optional `externalId`) — assumed credentials are cached, refreshed about 60 s before expiry, and concurrent requests share one refresh — or the default AWS credential chain
- Vertex AI with automatic private-key normalization
- Native Hugging Face and OpenRouter providers (OpenRouter no longer needs the OpenAI-compatible workaround)

##### Debug Payload Logging

`logging.types.aiLink.logUpstream` records the provider-native wire request/response bodies plus the upstream error body. All three payload flags (`logPrompt`, `logResponse`, `logUpstream`) can be overridden per target with `debug:`.

See [Logging](./config/logging.md).

##### Realtime Relay for Voice Agents

An authenticated, server-managed WebSocket relay to the OpenAI Realtime API on `/realtime` and `/proactions/realtime`, with admission control, session limits, backpressure, heartbeats, graceful shutdown, and per-modality token accounting.

See [Realtime Relay](./config/realtime.md).

##### Admin Panel

- New **Realtime** tab: relay status, live sessions, and aggregates by modality, duration and close category
- AI-Link tab gains **AI Targets** and **Guardrails** tables and a **fallback attempts** metric
- Dashboard shows AI-target selections and configured target/guardrail counts

See [Admin Panel](./admin-panel.md).

#### Improvements

- The legacy AI gateway subprocess is only started when at least one legacy `targets` entry exists; an `aiTargets`-only deployment never spawns it
- `/ready` reports the legacy gateway check (`ai_gateway`) as healthy with `skipped: true` when the gateway was intentionally not started
- Every request served by a legacy target logs one `warn`-level `aiLink` line naming the target id, so migration progress is visible without extra instrumentation
- `GET /` is public and returns the application name, version and description

#### Fixes

- Rate-limiting `windowMs` and `max` values coming from environment substitutions are coerced to integers, so windows now reset correctly
- Proxy upstream errors return a `502` JSON body (`Proxy error: …`) instead of a generic error

#### Infrastructure

- NestJS 12
- TypeScript 6
- pnpm pinned through `packageManager`
- UBI8 images on Node.js 22.23.2

#### Deprecations

- Legacy AI-Link `targets:` and their `config:` routing objects — migrate to `aiTargets`, see the [migration guide](./config/ai-link-migration.md)
- `hub.ailink_*` settings are only used while legacy targets exist

#### Documentation

- New: AI-Link Configuration (rewritten around `aiTargets`), Migration to `aiTargets`, Routing & Retries, Guardrails, Realtime Relay, Legacy targets
- Updated: overview, setup & configuration, admin panel, logging, operations guide, security hardening, voice agents guide, release notes

### 1.2.0 - 23/04/2026

#### New Features

##### Admin Panel

A new built-in browser-based admin panel for monitoring and diagnostics.

- **Secure authentication** — stateless API key authentication (`auth.admin.apiKeys`) independent of the main auth system; minimum 32-character key requirement; optional IP/CIDR allowlist via `auth.admin.allowedNetworks`
- **Dashboard** — process information, memory usage, configuration summary, and runtime usage charts
- **AI-Link metrics** — real-time token usage per provider/model, latency distributions, finish reason breakdowns, and per-action request tables
- **Proxy usage** — configured proxies and per-proxy request counts
- **MCP monitoring** — connection status, per-server latency/error metrics, and per-tool invocation counts
- **Config viewer** — full runtime configuration with sensitive fields automatically redacted
- **System tab** — Node.js version, uptime, and memory breakdown
- **Toolbar** — auto-refresh (30 s), manual refresh, statistics reset with confirmation, dark/light mode toggle

See [Admin Panel](./admin-panel.md) for full configuration and API reference.

##### AI-Link - SSE Streaming Support

The AI-Link gateway now natively proxies Server-Sent Events (SSE) streaming responses.

- Set `"stream": true` in the request body to receive a streaming response
- The Hub forwards the upstream SSE stream directly to the client with proper `text/event-stream` headers
- Client disconnects are detected and upstream connections are torn down cleanly to prevent resource leaks
- Streaming requests are tracked separately in the Admin Panel (streaming ratio metric)

##### MCP Gateway - Remote Server Support

- **HTTP Transport (`transport: http`):**
  - Connect to remote MCP servers via Streamable HTTP protocol
  - Uses HTTP POST for sending requests and Server-Sent Events (SSE) for receiving responses
  - Custom headers support for authentication (e.g., `Authorization: Bearer ...`)
  - Ideal for centralized MCP servers or third-party hosted services

- **WebSocket Transport (`transport: websocket`):**
  - Connect to remote MCP servers via WebSocket for low-latency, bidirectional communication
  - Suitable for real-time MCP servers requiring persistent connections

- **Backward Compatible:**
  - `stdio` transport remains the default for local subprocess-based MCP servers
  - Existing configurations continue to work without changes

##### MCP Gateway - Connection Stability Improvements

- **Configurable Timeouts:**
  - `connectTimeout`: Connection timeout in milliseconds (default: 30000ms)
  - `toolTimeout`: Tool invocation timeout in milliseconds (default: 60000ms)

- **Connection Reliability:**
  - Automatic transport cleanup on connection failure (prevents orphaned processes/connections)
  - Disconnect timeout protection during shutdown (prevents hanging on unresponsive servers)
  - Async connection drop detection via transport lifecycle handlers
  - Timer leak fixes in timeout handling

##### MCP Gateway - Server Log Routing

stderr output from `stdio` MCP server subprocesses is now captured and routed through the Hub's structured logger instead of spilling directly to the parent process console. Each line is emitted at `warn` level with the server name and `stream: stderr` metadata, making MCP server diagnostic output visible in your standard log pipeline (file rotation, JSON export, etc.).

#### Documentation

- Updated MCP Gateway documentation with remote transport configuration examples
- Added transport type comparison table
- Added security considerations for remote connections
- Added Admin Panel documentation
- Added streaming support to AI-Link documentation

### 1.1.0 - 11/11/2025

#### New Features

##### Enhanced AI Provider Support

- **9 New AI Providers Added:**
  - **LEPTON** - Lepton AI cloud platform
  - **KLUSTER_AI** - Kluster AI services
  - **NSCALE** - nScale AI infrastructure
  - **HYPERBOLIC** - Hyperbolic AI platform
  - **BYTEZ** - Bytez AI services
  - **FEATHERLESS_AI** - Featherless AI
  - **KRUTRIM** - Krutrim AI
  - **QDRANT** - Qdrant vector database AI
  - **THREE_ZERO_TWO_AI** - 302.AI platform

##### Custom AI-Link Configurations:

  - Added support for advanced configuration objects
  - Enables custom load balancing strategies and fallback configurations
  - Allows guardrail integration for content moderation
  - Provides flexibility for complex multi-provider setups

##### Custom OpenAI-Compatible Providers:

  - Integration support for any OpenAI-compatible API endpoint
  - Configurable endpoint URL with placeholder support (e.g., `{path}`)
  - Custom header management for provider-specific authentication

#### AI Gateway Reliability & Performance

- **Gateway Readiness Checks:**
  - Requests are now rejected with HTTP 503 if AI Gateway is not ready
  - Prevents request failures during gateway startup
  - Improves overall system reliability and user experience

- **Vertex AI Token Caching:**
  - Implemented intelligent token caching for Google Vertex AI
  - Reduces authentication overhead and improves response times
  - Automatic token refresh and lifecycle management

- **AI Request Timeout Configuration:**
  - Configurable timeout for AI requests to prevent hanging connections
  - Default timeout protection against unresponsive upstream services
  - Enhanced error details for timeout scenarios

#### Security Enhancements

- **Security Scanning Toolchain:**
  - Integrated comprehensive security scanning tools (Trivy, Gitleaks, Docker Scout)
  - Added npm scripts for vulnerability scanning workflows
  - Automated secret detection and CVE scanning
  - SBOM (Software Bill of Materials) generation support
- **Enhanced Session Cookie Security:**
  - Consolidated cookie configuration under unified `cookie` key
  - Added security defaults: `httpOnly`, `secure`, `sameSite` options
  - Improved cookie secret management with automatic fallback and warnings
  - Better protection against CSRF and session hijacking attacks
- **Configuration Security:**
  - Added validation options for stricter configuration checking
  - Improved configuration defaults for production deployments
  - Enhanced error messages for misconfigured security settings

#### Health Check Improvements

- **Health Check Guard:**
  - Added guard protection for health check endpoints
  - Configurable access control for monitoring endpoints
  - Supports deployment behind load balancers and orchestrators

#### Improvements

##### Error Handling & Diagnostics

- **Enriched HTTP Error Details:**
  - Enhanced error messages with upstream connection details
  - Better diagnostic information for debugging AI provider issues
  - Improved logging of request/response errors with context

- **Improved Error Logging:**
  - More detailed error traces for troubleshooting
  - Better filtering of sensitive data in error logs
  - Enhanced log filtering for circular references and complex objects

#### Logging Enhancements

- **Improved Sensitive Data Filtering:**
  - Enhanced detection of API keys, tokens, and passwords
  - Handles JSON-embedded credentials and circular object references
  - More robust redaction across all log types

- **Optimized Debug Logging:**
  - Reduced verbose debug output in production
  - Cleaned up unnecessary debug logs in tools and YouTube controllers
  - Better performance with selective logging

#### Prime Panel Updates

##### Status Indicator & Notifications

Visual feedback system for communicating process status and notifications to users.

###### API Functions:

- `window.ProActionsPanel.setStatus(statusText)` - Displays a status bar at the bottom of the panel with animated indicators to show ongoing processes
- `window.ProActionsPanel.clearStatus()` - Hides the status bar
- `window.ProActionsPanel.showNotification(message, duration?)` - Shows a temporary toast notification (default: 3 seconds)

#### Infrastructure

- **Node.js 22 Runtime:**
  - Continued use of Node.js 22 for optimal performance
  - Updated Red Hat UBI8 base image with Node.js 22
  - Enhanced stability and security with latest LTS features

### 1.0.9 - 02/07/2025

##### Maintenance

Updated ProActions Core and AI-Kit to version 1.0.12 for Prime Panel.

### 1.0.8 - 30/06/2025

#### Improvements

##### YouTube API Integration

- Added optional thumbnail upload to YouTube Video upload.

##### Maintenance

- Updated libraries and dependencies.

### 1.0.7 - 28/04/2025

We're excited to announce the release of ProActions Hub 1.0.7, featuring significant new capabilities and improvements to enhance your workflow integration with external services.

#### New Features

##### YouTube API Integration

- Added comprehensive YouTube API support for authentication and video upload
- Enables seamless content publishing directly to YouTube from within Swing workflows
- Supports standard OAuth authentication flow for secure access to YouTube accounts

##### Prime Panel (Beta)

- Introduced beta support for the new Prime Panel
- Allows ProActions workflows to be used directly from the Prime interface

##### JSON Logging

- Implemented structured JSON logging for improved log analysis and monitoring
- Enhances debugging capabilities and system observability
- This allows a direct integration with **Méthode Logging Platform**

##### TLS Encryption Support

- Added native TLS/SSL encryption for secure communication
- Ensures data transmitted between ProActions Hub and external services remains private
- Configurable through the config.yaml configuration file

##### Swagger Documentation

- Integrated Swagger UI for comprehensive API documentation
- Provides interactive exploration of ProActions Hub API endpoints

##### Expanded LLM Provider Support

- Added support for five new LLM/AI service providers:
  - Cortex
  - Nebius
  - Recraft-AI
  - Milvus
  - Replicate

#### Improvements

- Added the ability to customize the cookie name used for authentication

### 1.0.6 - 13/12/2024

#### New Features

- **Support for Additional AI Services**:
  - **xAI Grok API**: Added integration for xAI Grok, expanding the range of supported AI services.
  - **Lambda Labs**: Enabled support for workflows using Lambda Labs.
  - **DashScope**: Integrated DashScope, allowing seamless interaction with its capabilities.
  - **SageMaker**: Added support for AWS SageMaker, enhancing machine learning workflow options.

#### Improvements

- **Enhanced Error Handling**:
  - Improved handling of remote service errors by including the original error details in response messages. This provides better transparency and debugging information.

#### Maintenance

- **Docker Image Update**:
  - Updated the Docker image to use **Node.js v20**.

### 1.0.4 - 01/10/2024

- Support for additional providers
  - azure-ai
  - huggingface
  - deepseek
  - predibase
  - triton
  - voyage
  - github
  - deepbricks
  - siliconflow
  - cerebras
  - inference-net
  - sambanova
  - upstage

### 1.0.3 - 30/09/2024

- Fixed http communication issue with invalid responses.

### 1.0.2 - 20/09/2024

- Added private key authentication for Google Vertex AI.
