---
id: operations-guide
title: Operations Guide
sidebar_label: Operations Guide
sidebar_position: 9
---

# Operations Guide

This guide provides comprehensive operational procedures supporting ProActions Hub deployments.

## Overview

ProActions Hub is deployed as a containerized application using either Podman (RHEL 9+) or Docker (RHEL 7). The standard installation path is `/methode/methdig/tools/proactions-hub/`.

### Key Information

- **Default Port:** 3000 (API)
- **Configuration:** `/methode/methdig/tools/proactions-hub/config.yaml`
- **Logs:** Container logs via Podman/Docker, optional file logs in `logs/` directory
- **Health Checks:** `/health` (liveness), `/ready` (readiness)

## Service Management

### Podman Deployments (RHEL 9+)

#### Using Control Script

```bash
cd /methode/methdig/tools/proactions-hub/

# Start service
bash start_stop_proactionshub.bash start

# Stop service
bash start_stop_proactionshub.bash stop

# Restart service
bash start_stop_proactionshub.bash restart
```

#### Using Monit

```bash
# Start service
sudo monit start XXXX-YYY-methdig-worker_proactionshub

# Stop service
sudo monit stop XXXX-YYY-methdig-worker_proactionshub

# Restart service
sudo monit restart XXXX-YYY-methdig-worker_proactionshub

# Check status
sudo monit status XXXX-YYY-methdig-worker_proactionshub
```

:::info Monit Configuration
Replace `XXXX-YYY` with the actual server identifier from your monit configuration.
:::

#### Direct Podman Commands

```bash
# Check if running
podman ps | grep proactionshub

# Start container
podman start proactionshub

# Stop container
podman stop proactionshub

# Restart container
podman restart proactionshub

# View container details
podman inspect proactionshub
```

### Docker Deployments (RHEL 7)

#### Using Docker Compose

```bash
cd /methode/methdig/tools/proactions-hub/

# Start service
docker compose up -d

# Stop service
docker compose down

# Restart service
docker compose restart

# View status
docker compose ps
```

#### Direct Docker Commands

```bash
# Check if running
docker ps | grep proactionshub

# Start container
docker start proactionshub

# Stop container
docker stop proactionshub

# Restart container
docker restart proactionshub
```

## Health Monitoring

### Health Check Endpoints

**Liveness Check:**
```bash
curl http://localhost:$PROACTIONS_HUB_PORT/health
```

Expected response:
```json
{"status":"ok"}
```

**Readiness Check:**
```bash
curl http://localhost:$PROACTIONS_HUB_PORT/ready
```

Expected response:
```json
{"status":"ready"}
```

:::warning AI-Link Health
The `/ready` endpoint checks if the AI-Link service is healthy. A failure indicates the AI-Link subprocess is not responding.
:::

### Load Balancer Integration

Configure health checks in your load balancer:

- **Path:** `/health`
- **Interval:** 30 seconds
- **Timeout:** 5 seconds
- **Healthy threshold:** 2 consecutive successes
- **Unhealthy threshold:** 2 consecutive failures

### Quick Status Check

```bash
# Podman
podman healthcheck run proactionshub

# Check HTTP response
curl -s -o /dev/null -w "%{http_code}" http://localhost:$PROACTIONS_HUB_PORT/health
```

## Log Management

### Viewing Logs

**Podman:**
```bash
# View recent logs
podman logs proactionshub

# Follow logs in real-time
podman logs -f proactionshub

# View last 100 lines
podman logs --tail 100 proactionshub

# View logs since timestamp
podman logs --since 2024-01-15T10:00:00 proactionshub
```

**Docker:**
```bash
cd /methode/methdig/tools/proactions-hub/

# View recent logs
docker compose logs

# Follow logs in real-time
docker compose logs -f

# View specific service logs
docker compose logs proactionshub

# View last 100 lines
docker compose logs --tail 100
```

### Log Types

ProActions Hub generates several log types (if file logging is enabled):

```
logs/
├── app-access-YYYY-MM-DD.log     # HTTP access logs
├── app-error-YYYY-MM-DD.log      # Error logs
├── app-aiLink-YYYY-MM-DD.log     # AI-Link operations
├── app-proxy-YYYY-MM-DD.log      # Proxy operations
├── app-youtube-YYYY-MM-DD.log    # YouTube operations
└── app-tools-YYYY-MM-DD.log      # Tools operations
```

### Filtering Logs

**By log level (JSON format):**
```bash
podman logs proactionshub | jq 'select(.level == "error")'
```

**By module:**
```bash
podman logs proactionshub | jq 'select(.module == "aiLink")'
```

**By timestamp:**
```bash
podman logs --since 2024-01-15T10:00:00 proactionshub | grep ERROR
```

**Search for specific errors:**
```bash
podman logs proactionshub | grep "authentication failed"
```

### Log Analysis

**Count errors in last hour:**
```bash
podman logs --since 1h proactionshub | jq 'select(.level == "error")' | wc -l
```

**Token usage summary (AI-Link):**
```bash
cat logs/app-aiLink-*.log | jq -r '.totalTokens' | awk '{sum+=$1} END {print sum}'
```

**Most frequent errors:**
```bash
podman logs proactionshub | jq -r 'select(.level == "error") | .message' | sort | uniq -c | sort -rn | head -10
```

## Updating ProActions Hub

### Podman Update Procedure

```bash
# 1. Login to Artifactory
podman login artifactory.eidosmedia.sh/docker-releases -u <username>

# 2. Navigate to installation directory
cd /methode/methdig/tools/proactions-hub/

# 3. Check current version (optional)
podman inspect proactionshub | jq '.[0].Config.Labels'

# 4. Update version if necessary
vi ~/cfg/env/proactionshub.vars
# Set: export PROACTIONS_HUB_VERSION=latest

# 5. Reload environment variables
source ~/.bashrc

# 6. Pull latest image
podman pull artifactory.eidosmedia.sh/docker-releases/proactions/proactions-hub:$PROACTIONS_HUB_VERSION

# 7. Recreate the pod
bash proactionshub.bash

# 8. Stop and remove existing container
bash start_stop_proactionshub.bash stop

# 9. Start new container
bash start_stop_proactionshub.bash start

# 10. Verify service is running
curl http://localhost:$PROACTIONS_HUB_PORT/health

# 11. Check logs for errors
podman logs proactionshub

# 12. Logout from Artifactory
podman logout artifactory.eidosmedia.sh/docker-releases
```

### Docker Update Procedure

```bash
# 1. Login to Artifactory
docker login artifactory.eidosmedia.sh/docker-releases -u <username>

# 2. Navigate to installation directory
cd /methode/methdig/tools/proactions-hub/

# 3. Pull latest images
docker compose pull

# 4. Recreate containers with new images
docker compose up --force-recreate --build -d

# 5. Verify service is running
curl http://localhost:$PROACTIONS_HUB_PORT/health

# 6. Check logs for errors
docker compose logs

# 7. Logout from Artifactory
docker logout artifactory.eidosmedia.sh/docker-releases
```

### Rollback Procedure

**Podman:**
```bash
# 1. Pull previous version
podman pull artifactory.eidosmedia.sh/docker-releases/proactions/proactions-hub:<previous-version>

# 2. Update version variable
export PROACTIONS_HUB_VERSION=<previous-version>

# 3. Restart with previous version
bash start_stop_proactionshub.bash stop
bash start_stop_proactionshub.bash start

# 4. Verify
curl http://localhost:$PROACTIONS_HUB_PORT/health
```

**Docker:**
```bash
# 1. Update image tag in docker-compose.yml
vi docker-compose.yml
# Change: image: artifactory.eidosmedia.sh/docker-releases/proactions/proactions-hub:<previous-version>

# 2. Pull and recreate
docker compose pull
docker compose up --force-recreate -d

# 3. Verify
curl http://localhost:$PROACTIONS_HUB_PORT/health
```

## Troubleshooting

### Service Won't Start

**Symptoms:** Container fails to start or exits immediately

**Investigation:**
```bash
# Check container logs
podman logs proactionshub

# Check container status
podman ps -a | grep proactionshub

# Inspect container
podman inspect proactionshub
```

**Common Causes:**

1. **Port already in use:**
   ```bash
   # Check what's using the port
   netstat -tulpn | grep $PROACTIONS_HUB_PORT
   # or
   lsof -i :$PROACTIONS_HUB_PORT
   ```

2. **Configuration errors:**
   ```bash
   # Validate YAML syntax
   yamllint config.yaml

   # Check for missing environment variables
   podman logs proactionshub | grep "Configuration validation failed"
   ```

3. **Volume mount issues:**
   ```bash
   # Check volume mounts
   podman inspect proactionshub | jq '.[0].Mounts'

   # Verify file permissions
   ls -la config.yaml
   ls -la tokens/
   ```

**Solutions:**
- Stop conflicting services on ports $PROACTIONS_HUB_PORT
- Fix configuration errors identified in logs
- Ensure config file and volumes are accessible
- Check SELinux labels (use `:Z` flag)

### Health Check Failing

**Symptoms:** `/health` or `/ready` returns non-200 status

**Investigation:**
```bash
# Test health endpoint
curl -v http://localhost:$PROACTIONS_HUB_PORT/health

# Test readiness endpoint
curl -v http://localhost:$PROACTIONS_HUB_PORT/ready

# Check logs for AI-Link errors
podman logs proactionshub | grep -i "ailink\|ai-link"
```

**Common Causes:**

1. **AI-Link subprocess not starting:**
   - Check logs for AI-Link startup errors
   - Verify Node.js is available in container
   - Check ailink_port is not in use

2. **Port binding issues:**
   - Verify ports are exposed correctly
   - Check firewall rules

**Solutions:**
```bash
# Restart service
bash start_stop_proactionshub.bash restart

# Check AI-Link health check configuration
cat config.yaml | grep ailink_health_check

# Verify both ports are accessible
netstat -tulpn | grep -E "$PROACTIONS_HUB_PORT"
```

### Authentication Issues

**Symptoms:** 401 Unauthorized errors

**Investigation:**
```bash
# Check authentication configuration
cat config.yaml | grep -A 10 "^auth:"

# Test with valid credentials
curl -v http://localhost:$PROACTIONS_HUB_PORT/v1/chat/completions \
  -H "Authorization: Bearer <token>"

# Check session store (Redis)
redis-cli ping
```

**Common Causes:**

1. **EDAPI unreachable:**
   ```bash
   # Test EDAPI connectivity
   curl -v $EDAPI_URL
   ```

2. **Session store issues:**
   ```bash
   # Check Redis connectivity
   redis-cli -u $PROACTIONS_REDIS_URL ping
   ```

3. **Invalid credentials:**
   - Verify tokens/passwords are correct
   - Check EDAPI user has proper permissions

**Solutions:**
- Verify EDAPI URL is accessible
- Restart Redis if connection issues
- Validate authentication credentials
- Check session cookie configuration

### AI-Link Request Failures

**Symptoms:** 502 Bad Gateway or timeout errors on AI endpoints

**Investigation:**
```bash
# Check AI-Link logs
podman logs proactionshub | grep aiLink

# Test specific target
curl -X POST http://localhost:$PROACTIONS_HUB_PORT/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "x-target: openai" \
  -d '{"messages": [{"role": "user", "content": "test"}]}'

# Check target configuration
cat config.yaml | grep -A 10 "id: openai"
```

**Common Causes:**

1. **Invalid API key:**
   - Check environment variable is set
   - Verify API key with provider

2. **Provider-specific configuration missing:**
   ```bash
   # For Azure OpenAI, check:
   cat config.yaml | grep -A 5 "azureDeploymentId"
   ```

3. **Network connectivity:**
   ```bash
   # Test provider connectivity from container
   podman exec proactionshub curl -v https://api.openai.com
   ```

4. **Request timeout:**
   - Check if timeout is too short for the model
   - Review `requestTimeout` configuration

**Solutions:**
- Verify API keys are valid and set correctly
- Add missing provider-specific configuration
- Check network connectivity to AI providers
- Increase timeout for slow models
- Review AI provider quotas and limits

### High Memory/CPU Usage

**Symptoms:** Container using excessive resources

**Investigation:**
```bash
# Check resource usage
podman stats proactionshub

# Check for memory leaks
podman exec proactionshub node --heap-size

# Review recent requests
podman logs --tail 1000 proactionshub | grep -E "duration|responseTime"
```

**Common Causes:**

1. **High request volume:**
   - Check access logs for traffic patterns
   - Verify rate limiting is enabled

2. **Large AI requests:**
   - Check token usage in AI-Link logs
   - Review max_tokens configuration

3. **Memory leak:**
   - Check container uptime
   - Review application logs for errors

**Solutions:**
- Enable rate limiting to control traffic
- Set appropriate max_tokens limits
- Restart container to clear memory
- Review and optimize AI request patterns
- Consider vertical scaling (more RAM/CPU)

### Disk Space Issues

**Symptoms:** Errors writing logs or tokens

**Investigation:**
```bash
# Check disk usage
df -h

# Check log directory size
du -sh /methode/methdig/tools/proactions-hub/logs/

# Find large files
find /methode/methdig/tools/proactions-hub/ -type f -size +100M
```

**Solutions:**
```bash
# Clean old logs
find logs/ -name "*.log" -mtime +30 -delete

# Archive old logs
tar -czf logs-archive-$(date +%Y%m%d).tar.gz logs/*.log
find logs/ -name "*.log" -mtime +7 -delete

# Configure log rotation
# Edit config.yaml:
# logging:
#   fileRotation: true
#   maxFileSize: 10m
#   maxFiles: 14d
```

## Monitoring and Alerting

### Key Metrics to Monitor

| Metric | Threshold | Action |
|--------|-----------|--------|
| Health check failures | 2 consecutive | Alert + investigate |
| Error rate | >5% of requests | Alert + review logs |
| Response time | >5 seconds average | Investigate performance |
| CPU usage | >80% sustained | Consider scaling |
| Memory usage | >80% sustained | Check for leaks |
| Disk usage | >80% | Clean logs, increase capacity |
| Token usage | Approaching quota | Alert stakeholders |

### Alert Configuration

**Monit Example:**
```
check process proactionshub with pidfile /var/run/proactionshub.pid
  start program = "/usr/local/bin/start_proactionshub.sh"
  stop program = "/usr/local/bin/stop_proactionshub.sh"
  if failed host localhost port $PROACTIONS_HUB_PORT protocol http
    request "/health"
    with timeout 10 seconds
    for 2 cycles
  then restart
  if cpu > 80% for 5 cycles then alert
  if memory > 80% for 5 cycles then alert
```

**Custom Health Check Script:**
```bash
#!/bin/bash
# health-check.sh

HEALTH_URL="http://localhost:$PROACTIONS_HUB_PORT/health"
RESPONSE=$(curl -s -o /dev/null -w "%{http_code}" $HEALTH_URL)

if [ "$RESPONSE" != "200" ]; then
  echo "CRITICAL: Health check failed with status $RESPONSE"
  exit 2
fi

echo "OK: ProActions Hub is healthy"
exit 0
```

## Backup and Recovery

### Configuration Backup

```bash
# Backup configuration
tar -czf proactions-config-$(date +%Y%m%d).tar.gz \
  config.yaml \
  .env \
  docker-compose.yml

# Backup to remote location
scp proactions-config-*.tar.gz backup-server:/backups/proactions/
```

### Token Backup

```bash
# Backup YouTube tokens
tar -czf proactions-tokens-$(date +%Y%m%d).tar.gz tokens/

# Secure backup with encryption
tar -czf - tokens/ | openssl enc -aes-256-cbc -out proactions-tokens-$(date +%Y%m%d).tar.gz.enc
```

### Recovery Procedure

```bash
# 1. Restore configuration
cd /methode/methdig/tools/proactions-hub/
tar -xzf proactions-config-YYYYMMDD.tar.gz

# 2. Restore tokens (if applicable)
tar -xzf proactions-tokens-YYYYMMDD.tar.gz

# 3. Set proper permissions
chmod 600 config.yaml
chmod 600 .env
chmod 600 tokens/*.json

# 4. Restart service
bash start_stop_proactionshub.bash restart

# 5. Verify
curl http://localhost:$PROACTIONS_HUB_PORT/health
```

## Performance Optimization

### Configuration Tuning

**Optimize HTTP client timeouts:**
```yaml
hub:
  http_client:
    timeout: 120000  # 2 minutes for most models
    maxRedirects: 5
```

**Configure appropriate request timeouts per target:**
```yaml
targets:
  - id: fast-model
    requestTimeout: 30000  # 30 seconds

  - id: slow-model
    requestTimeout: 300000  # 5 minutes
```

### Resource Limits

**Podman resource constraints:**
```bash
podman run --memory=2g --cpus=2 proactions-hub
```

**Docker Compose:**
```yaml
services:
  proactions-hub:
    deploy:
      resources:
        limits:
          memory: 2G
          cpus: '2'
```

## Common Issues Reference

| Issue | Symptoms | Quick Fix |
|-------|----------|-----------|
| Service not starting | Container exits immediately | Check logs, verify configuration |
| Authentication failing | 401 errors | Verify EDAPI URL, check session store |
| AI requests failing | 502 errors | Verify API keys, check provider connectivity |
| Health check failing | `/ready` returns 503 | Restart service, check AI-Link logs |
| High memory usage | Container OOM killed | Restart container, review request patterns |
| Disk space full | Write errors in logs | Clean old logs, configure rotation |
| YouTube auth failing | OAuth errors | Verify redirect URI, check client credentials |
| Slow responses | High response times | Check AI provider status, review timeouts |

## Appendix: Quick Reference

### Essential Commands

```bash
# Status
curl http://localhost:$PROACTIONS_HUB_PORT/health
podman ps | grep proactionshub

# Logs
podman logs -f proactionshub

# Restart
bash start_stop_proactionshub.bash restart

# Update
podman pull artifactory.eidosmedia.sh/docker-releases/proactions/proactions-hub:latest
bash start_stop_proactionshub.bash restart
```

### File Locations

```
/methode/methdig/tools/proactions-hub/
├── config.yaml           # Main configuration
├── .env                         # Environment variables
├── tokens/                      # YouTube OAuth tokens
├── logs/                        # Log files (if enabled)
├── start_stop_proactionshub.bash  # Control script
└── docker-compose.yml           # Docker Compose config (Docker only)
```

### Environment Variables

```bash
PROACTIONS_HUB_PORT=3000
PROACTIONS_HUB_AILINK_PORT=3001
EDAPI_URL=https://edapi.example.com/edapi
PROACTIONS_SESSION_SECRET=<secret>
OPENAI_API_KEY=sk-...
AZURE_OPENAI_API_KEY=...
```
