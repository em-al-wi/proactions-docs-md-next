---
id: security-hardening
title: Security Hardening
sidebar_label: Security Hardening
sidebar_position: 8
---

# Security Hardening

This guide covers security best practices for deploying ProActions Hub in production environments serving publishing houses and financial institutions.

## Overview

ProActions Hub includes multiple security layers:

1. **Authentication** - EDAPI or token-based access control
2. **Session Management** - Secure session storage and cookies
3. **TLS/HTTPS** - Encrypted transport
4. **Security Headers** - Helmet middleware protection
5. **CORS** - Cross-origin request control
6. **Rate Limiting** - DoS protection
7. **Request Validation** - Body size limits
8. **Credential Management** - Environment variable protection
9. **Logging Security** - Sensitive data filtering

## Authentication

### EDAPI Authentication (Recommended)

EDAPI provides enterprise-grade authentication integration:

```yaml
auth:
  enabled: true
  type: edapi
  edapi_url: ${EDAPI_URL}
  session:
    store: redis  # Use Redis in production
    cookie:
      secret: ${PROACTIONS_SESSION_SECRET}
      secure: true
      httpOnly: true
      sameSite: strict
      maxAge: 86400000  # 24 hours
    redis_client:
      url: ${PROACTIONS_REDIS_URL}
```

**Environment variables:**
```bash
EDAPI_URL=https://your-edapi-instance/edapi
PROACTIONS_SESSION_SECRET=<generate-random-secret>
PROACTIONS_REDIS_URL=redis://localhost:6379
```

:::tip Generate Secret
Generate a strong session secret:
```bash
openssl rand -base64 32
```
:::

### Token-Based Authentication

For development or non-EDAPI environments:

```yaml
auth:
  enabled: true
  type: token
  tokens:
    - ${AUTH_TOKEN_1}
    - ${AUTH_TOKEN_2}
  session:
    store: memory
    cookie:
      secret: ${PROACTIONS_SESSION_SECRET}
```

**Environment variables:**
```bash
AUTH_TOKEN_1=<strong-random-token-1>
AUTH_TOKEN_2=<strong-random-token-2>
```

:::warning Development Only
Token-based auth is less secure than EDAPI. Use only for development or testing.
:::

### Public Endpoints

Some endpoints are intentionally public (no authentication required):

- `/youtube/auth/google/callback` - OAuth callback handler
- `/health` - Health check endpoint
- `/ready` - Readiness check endpoint

All other endpoints require authentication when `auth.enabled: true`.

## Session Management

### Session Store

**Production:** Use Redis for session storage

```yaml
auth:
  session:
    store: redis
    redis_client:
      url: ${PROACTIONS_REDIS_URL}
      # Optional: TLS configuration
      tls:
        rejectUnauthorized: true
        ca: ${REDIS_CA_CERT}
```

**Development:** Memory store (not for production)

```yaml
auth:
  session:
    store: memory
```

### Cookie Configuration

Secure cookie settings for production:

```yaml
auth:
  session:
    cookie:
      name: proactions_session        # Custom cookie name
      secret: ${SESSION_SECRET}        # Strong random secret
      httpOnly: true                   # Prevent XSS access
      secure: true                     # HTTPS only
      sameSite: strict                 # CSRF protection
      maxAge: 86400000                 # 24 hours (in ms)
```

**Cookie Security Attributes:**

| Attribute | Value | Purpose |
|-----------|-------|---------|
| `httpOnly` | `true` | Prevents JavaScript access (XSS protection) |
| `secure` | `true` | Sends cookie only over HTTPS |
| `sameSite` | `strict` | Prevents CSRF attacks |
| `maxAge` | `86400000` | Session timeout (24 hours) |

:::info Auto-Detection
If `secure` is not specified, it's automatically set to `true` when TLS is enabled or `trustProxy` is configured.
:::

## TLS/HTTPS Configuration

### Enable TLS

```yaml
hub:
  tls:
    enabled: true
    cert: ${TLS_CERT_PATH}
    key: ${TLS_KEY_PATH}
    ca: ${TLS_CA_PATH}
    passphrase: ${TLS_PASSPHRASE}  # If key is encrypted
    rejectUnauthorized: true
    ciphers: TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
```

**Environment variables:**
```bash
TLS_CERT_PATH=/etc/ssl/certs/proactions.crt
TLS_KEY_PATH=/etc/ssl/private/proactions.key
TLS_CA_PATH=/etc/ssl/certs/ca-bundle.crt
TLS_PASSPHRASE=<key-passphrase>
```

### Volume Mounts for Certificates

**Docker Compose:**
```yaml
volumes:
  - /etc/ssl/certs:/etc/ssl/certs:ro
  - /etc/ssl/private:/etc/ssl/private:ro
```

**Podman:**
```bash
-v /etc/ssl/certs:/etc/ssl/certs:ro,Z \
-v /etc/ssl/private:/etc/ssl/private:ro,Z
```

### Certificate Recommendations

1. **Use valid certificates** - From trusted CA (Let's Encrypt, DigiCert, etc.)
2. **Enable strong ciphers** - TLS 1.2+ with modern cipher suites
3. **Regular renewal** - Automate certificate renewal
4. **Protect private keys** - Restrictive file permissions (600)

## Reverse Proxy / Load Balancer

### Trust Proxy Configuration

When behind a reverse proxy or load balancer:

```yaml
hub:
  security:
    trustProxy: true
```

**Trust specific number of hops:**
```yaml
hub:
  security:
    trustProxy: 1  # Trust first proxy only
```

**Trust specific IPs:**
```yaml
hub:
  security:
    trustProxy:
      - 10.0.0.0/8
      - 172.16.0.0/12
      - 192.168.0.0/16
```

:::warning Security Risk
Only enable `trustProxy` when actually behind a proxy. Otherwise, clients can spoof IP addresses through `X-Forwarded-For` headers.
:::

### Proxy Headers

Ensure your reverse proxy sets these headers:

```
X-Forwarded-For: <client-ip>
X-Forwarded-Proto: https
X-Forwarded-Host: <original-host>
```

## Security Headers (Helmet)

Helmet provides security headers to protect against common attacks:

```yaml
hub:
  security:
    helmet:
      enabled: true
      contentSecurityPolicy: false  # Configure based on needs
      crossOriginEmbedderPolicy: true
      crossOriginOpenerPolicy: true
      crossOriginResourcePolicy: true
      dnsPrefetchControl: true
      frameguard: true
      hidePoweredBy: true
      hsts: true
      ieNoOpen: true
      noSniff: true
      originAgentCluster: true
      permittedCrossDomainPolicies: true
      referrerPolicy: true
      xssFilter: true
```

### Key Security Headers

**HTTP Strict Transport Security (HSTS):**
```yaml
hsts:
  maxAge: 31536000  # 1 year
  includeSubDomains: true
  preload: true
```

**Content Security Policy:**
```yaml
contentSecurityPolicy:
  directives:
    defaultSrc: ["'self'"]
    scriptSrc: ["'self'", "'unsafe-inline'"]
    styleSrc: ["'self'", "'unsafe-inline'"]
    imgSrc: ["'self'", "data:", "https:"]
```

**Frame Options:**
```yaml
frameguard:
  action: deny  # or 'sameorigin'
```

## CORS Configuration

### Restrictive CORS (Recommended)

```yaml
hub:
  security:
    cors:
      enabled: true
      origin: https://your-app.example.com
      methods: GET,POST,PUT,DELETE
      credentials: true
      allowedHeaders: Content-Type,Authorization,x-target
      maxAge: 86400
```

### Multiple Origins

```yaml
hub:
  security:
    cors:
      origin:
        - https://app1.example.com
        - https://app2.example.com
```

### Development CORS

```yaml
hub:
  security:
    cors:
      enabled: true
      origin: true  # Reflects request origin
```

:::danger Production Warning
Never use `origin: true` or `origin: '*'` in production. Always specify allowed origins explicitly.
:::

## Rate Limiting

Protect against DoS attacks with rate limiting:

```yaml
hub:
  security:
    trustProxy: true  # Required for accurate IP detection
    rateLimit:
      enabled: true
      windowMs: 900000      # 15 minutes
      max: 100              # 100 requests per window per IP
      message: Too many requests from this IP, please try again later.
      standardHeaders: true
      legacyHeaders: false
```

### Rate Limit Headers

Clients receive rate limit information:

```
RateLimit-Limit: 100
RateLimit-Remaining: 95
RateLimit-Reset: 1642345678
```

### Customizing Rate Limits

**Higher limits for trusted clients:**
Consider implementing custom rate limiting logic with IP whitelist.

**Different limits per endpoint:**
Requires custom implementation (contact development team).

## Request Body Limits

Prevent oversized payloads:

```yaml
hub:
  security:
    limits:
      json: 50mb       # Max JSON payload size
      urlencoded: 10mb # Max URL-encoded payload size
```

**Recommendations:**
- **API endpoints:** 1-10MB
- **File uploads:** 50-100MB
- **Video uploads:** 500MB-2GB (YouTube)

## Credential Management

### Environment Variables

**Always use environment variables for secrets:**

```yaml
# Good
apiKey: ${OPENAI_API_KEY}
edapi_url: ${EDAPI_URL}

# Bad - Never do this
apiKey: sk-abc123xyz
edapi_url: https://edapi.example.com
```

### Secret Storage

**Development:**
```bash
# .env file (never commit to git)
OPENAI_API_KEY=sk-abc123
EDAPI_URL=https://edapi.example.com
```

**Production:**
- Use container secrets (Docker/Podman secrets)
- Use secret management services (AWS Secrets Manager, HashiCorp Vault)
- Use environment variables from secure storage

### File Permissions

Protect sensitive files:

```bash
# Configuration file
chmod 600 config/config.yaml

# Environment file
chmod 600 .env

# Token files
chmod 600 tokens/*.json

# TLS private keys
chmod 600 /etc/ssl/private/proactions.key
```

## Logging Security

### Sensitive Data Filtering

Ensure sensitive data filtering is enabled:

```yaml
logging:
  sensitiveData:
    - password
    - token
    - apiKey
    - authorization
    - key
    - secret
    - sessionId
    - cookie
```

### Disable Stack Traces in Production

```yaml
logging:
  logTrace: false
```

### Secure Log Files

```bash
# Restrict log file access
chmod 640 logs/*.log

# Ensure logs directory is protected
chmod 750 logs/
```

## Network Security

### Internal Services

**AI-Link Port:** Never expose externally

```yaml
hub:
  ailink_port: 3001
  ailink_host: localhost  # Bind to localhost only
```

### Firewall Rules

Only expose necessary ports:

```bash
# Allow HTTPS only
firewall-cmd --add-port=443/tcp --permanent

# Block AI-Link port
firewall-cmd --remove-port=3001/tcp --permanent

# Reload firewall
firewall-cmd --reload
```

### Container Network Isolation

Use dedicated networks for containers:

**Docker Compose:**
```yaml
networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true

services:
  proactions-hub:
    networks:
      - frontend
      - backend
```

## API Security

### API Key Rotation

Regularly rotate API keys:

1. Generate new API key from provider
2. Update environment variable
3. Restart ProActions Hub
4. Revoke old API key

### Least Privilege

Configure minimal necessary scopes:

**YouTube OAuth:**
```yaml
# Minimal scope
scope: https://www.googleapis.com/auth/youtube.upload

# Full access (avoid if possible)
scope: https://www.googleapis.com/auth/youtube
```

### API Key Storage

**Never log API keys:**
```yaml
logging:
  sensitiveData:
    - apiKey
    - api_key
```

## Container Security

### Run as Non-Root User

ProActions Hub container runs as non-root user by default (good).

Verify:
```bash
podman inspect proactionshub | jq '.[0].Config.User'
```

### SELinux (RHEL/CentOS)

Enable SELinux labels for volumes:

```bash
# Podman with SELinux
-v ./config:/usr/src/app/dist/config:ro,Z
-v ./tokens:/usr/src/app/tokens:rw,Z
```

### Read-Only Root Filesystem

Mount configuration as read-only:

```bash
-v ./config/config.yaml:/usr/src/app/dist/config.yaml:ro
```

## Compliance Considerations

### Data Residency

Configure providers with appropriate regions:

```yaml
targets:
  - id: azure-eu
    provider: azure-openai
    azureRegion: westeurope
```

### Audit Logging

Enable comprehensive logging:

```yaml
logging:
  types:
    access:
      enabled: true
      file: true
    aiLink:
      enabled: true
      file: true
    error:
      enabled: true
      file: true
```

### Data Retention

Configure appropriate retention:

```yaml
logging:
  maxFiles: 365d  # 1 year for compliance
```

### PII Protection

- Enable sensitive data filtering in logs
- Don't enable `logPrompt` or `logResponse` with PII data
- Implement data masking in application layer

## Security Checklist

### Pre-Production

- [ ] Authentication enabled (EDAPI preferred)
- [ ] Strong session secret generated
- [ ] TLS/HTTPS enabled with valid certificates
- [ ] Trust proxy configured if behind load balancer
- [ ] Helmet security headers enabled
- [ ] CORS configured with specific origins
- [ ] Rate limiting enabled
- [ ] Request body limits set
- [ ] Sensitive data filtering enabled
- [ ] Stack traces disabled in logs
- [ ] API keys stored in environment variables
- [ ] File permissions restricted
- [ ] Only necessary ports exposed
- [ ] Container running as non-root

### Post-Deployment

- [ ] Verify HTTPS is working
- [ ] Test authentication
- [ ] Verify rate limiting
- [ ] Check security headers (use securityheaders.com)
- [ ] Review access logs
- [ ] Test CORS policy
- [ ] Verify sensitive data is filtered in logs
- [ ] Monitor for security events
- [ ] Regular security updates applied

## Security Best Practices

### Configuration

1. **Enable authentication** - Never run without auth in production
2. **Use EDAPI** - Prefer enterprise authentication over tokens
3. **Enable TLS** - Always use HTTPS in production
4. **Configure Helmet** - Enable all appropriate security headers
5. **Restrict CORS** - Specify allowed origins explicitly

### Credentials

1. **Environment variables** - Never hardcode secrets
2. **Secret rotation** - Rotate API keys regularly
3. **Least privilege** - Use minimal necessary scopes
4. **Secret storage** - Use proper secret management tools
5. **File permissions** - Restrict access to sensitive files

### Network

1. **Expose minimal ports** - Only expose main API port
2. **Trust proxy carefully** - Only when actually behind proxy
3. **Use firewalls** - Block unnecessary ports
4. **Network isolation** - Use container networks
5. **Internal services** - Keep AI-Link port internal

### Monitoring

1. **Enable logging** - Comprehensive audit trail
2. **Filter sensitive data** - Prevent credential leaks
3. **Monitor access logs** - Detect suspicious activity
4. **Alert on errors** - Unusual error rates
5. **Regular reviews** - Review security logs regularly

### Updates

1. **Stay current** - Apply security updates promptly
2. **Test updates** - Verify in staging first
3. **Monitor advisories** - Subscribe to security bulletins
4. **Scan regularly** - Automated vulnerability scanning
5. **Dependency updates** - Keep dependencies current

## Incident Response

### Security Event Detection

Monitor logs for:
- Repeated authentication failures
- Unusual request patterns
- Rate limit violations
- Error spikes
- Unauthorized access attempts

### Response Procedures

1. **Identify the issue** - Review logs and metrics
2. **Contain the threat** - Block malicious IPs, disable compromised accounts
3. **Investigate** - Determine scope and impact
4. **Remediate** - Fix vulnerabilities, rotate credentials
5. **Document** - Record incident details and response
6. **Review** - Post-mortem and preventive measures

## Next Steps

- [Operations Guide](operations-guide.md) for monitoring and maintenance
- Review your organization's security policies
- Conduct security assessment
- Implement monitoring and alerting
- Schedule regular security reviews
