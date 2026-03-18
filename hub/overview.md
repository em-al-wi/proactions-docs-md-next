---
id: overview
title: Overview
sidebar_label: Overview
sidebar_position: 1
---

# Overview

## Introduction

ProActions Hub is a REST API server that provides a unified gateway for accessing multiple AI services used by publishing houses and financial institutions. It serves as the backend infrastructure for the ProActions ecosystem, offering enterprise-grade features for AI integration, content processing, and secure API management.

## What is ProActions Hub?

ProActions Hub acts as a normalization layer and secure gateway between your applications and various AI service providers. It consolidates API key management, provides unified interfaces for diverse AI services, and adds enterprise security controls including authentication, logging, and compliance features.

## Key Capabilities

### AI Service Integration (AI-Link)

The AI-Link module provides a unified gateway to multiple AI providers including:

- **OpenAI** - GPT models and image generation
- **Azure OpenAI** - Microsoft-hosted OpenAI services
- **Anthropic** - Claude models
- **AWS Bedrock** - Amazon's AI service platform
- **Google Vertex AI / Gemini** - Google's AI services
- **50+ Additional Providers** - Including Perplexity, OpenRouter, Mistral, Groq, and more

All providers are accessed through consistent OpenAI-compatible endpoints (`/v1/chat/completions`, `/v1/images/generations`, etc.), with the AI-Link handling provider-specific authentication and request formatting.

### HTTP(s) Proxy

Forward HTTPs requests to configured targets with:

- Dynamic path rewriting
- Custom header management
- Support for multiple proxy destinations (DeepL, OpenAI, Azure OpenAI, etc.)

### YouTube Integration

OAuth 2.0-based YouTube API integration supporting:

- Multi-account management
- Video uploads with metadata and thumbnails
- Token storage and automatic refresh

### Content Extraction Tools

Built-in tools for content processing:

- URL content extraction
- Extensible architecture for additional tools

### MCP Gateway

Host and connect to [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) servers, enabling:

- Access to backend tools, databases, and filesystems
- Dynamic execution of Node.js and Python-based MCP servers
- Standardized API for tool invocation

### Enterprise Security

Production-ready security features:

- EDAPI or token-based authentication
- Session management (memory or Redis-backed)
- HTTPS/TLS support
- Helmet security headers
- CORS configuration
- Rate limiting
- Request body size limits
- Sensitive data filtering in logs

### Comprehensive Logging

Winston-based logging system with:

- Multiple log types (access, error, AI-Link, proxy, YouTube, tools)
- JSON or text output formats
- Configurable log levels per module
- Token usage tracking for cost monitoring
- Automatic sensitive data filtering
- File rotation support

## Architecture

The application consists of modular components:

- AI-Link Module (unified AI gateway)
- Proxy Module (HTTP forwarding)
- YouTube Module (OAuth and uploads)
- Tools Module (content extraction)
- MCP Module (tool gateway)
- Logging Module (centralized logging)
- Health Module (monitoring endpoints)
- Common Module (shared guards, providers, middleware)

## Use Cases

### Publishing Workflows

- Automated content generation and editing
- Multi-language translation
- Content extraction from web sources
- Video publishing to YouTube

### Financial Services

- Secure AI-powered analysis and reporting
- Document processing and extraction
- Compliance-focused logging and audit trails
- Centralized API key management

### Enterprise AI Integration

- Single gateway for multiple AI providers
- Provider failover and load balancing
- Token usage monitoring and cost control
- EDAPI-based authentication for existing systems

## Deployment Models

ProActions Hub can be deployed using:

- **Podman** (recommended for RHEL 9+)
- **Docker** (RHEL 7 with docker-compose)
- **Kubernetes** (container orchestration)

Standard installation path: `/methode/methdig/tools/proactions-hub/`

## What's Next?

This documentation is organized into the following sections:

1. **Setup & Initial Configuration** - Get started with your deployed container
2. **Configuration Guides** - Detailed configuration for each module
3. **Security Hardening** - Best practices for production deployments
4. **Operations Guide** - Maintenance, monitoring, and troubleshooting

:::info Prerequisites
This documentation assumes you have already deployed ProActions Hub in a Docker or Podman environment. For installation procedures, please consult your deployment documentation or contact your system administrator.
:::
