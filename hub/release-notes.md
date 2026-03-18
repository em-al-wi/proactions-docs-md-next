---
id: pah-release-notes
title: ProActions Hub Release Notes
sidebar_label: Hub Release Notes
---

### 1.2.0 - 05/02/2026

#### New Features

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

#### Documentation

- Updated MCP Gateway documentation with remote transport configuration examples
- Added transport type comparison table
- Added security considerations for remote connections

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
