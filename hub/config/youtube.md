---
id: youtube
title: YouTube Configuration
sidebar_label: YouTube
sidebar_position: 5
---

# YouTube Configuration

The YouTube module provides OAuth 2.0 authentication and video upload capabilities for one or more YouTube accounts. This guide covers setup, configuration, and usage.

## Quick Start

### Prerequisites

Before configuring YouTube integration, you need:

1. **Google Cloud Console project** with YouTube Data API v3 enabled
2. **OAuth 2.0 credentials** (Client ID and Client Secret)
3. **Redirect URI** configured in Google Cloud Console
4. **Persistent storage** for OAuth tokens

### Minimal Configuration

```yaml
youtube:
  accounts:
    - id: main
      client_id: ${YOUTUBE_MAIN_CLIENT_ID}
      client_secret: ${YOUTUBE_MAIN_CLIENT_SECRET}
      redirect_uri: ${YOUTUBE_MAIN_REDIRECT_URI}
  token_storage_path: ./tokens
  token_storage_file: youtube_token_{id}.json
```

Set environment variables:
```bash
YOUTUBE_MAIN_CLIENT_ID=123456789.apps.googleusercontent.com
YOUTUBE_MAIN_CLIENT_SECRET=your-client-secret
YOUTUBE_MAIN_REDIRECT_URI=http://localhost:${PROACTIONS_HUB_PORT}/youtube/auth/google/callback?target=main
```

:::warning Redirect URI
The redirect URI must match exactly what is configured in Google Cloud Console, including the `target` query parameter.
:::

## Google Cloud Console Setup

### Step 1: Create Project

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select an existing one
3. Note the project ID

### Step 2: Enable YouTube Data API v3

1. Navigate to **APIs & Services** > **Library**
2. Search for "YouTube Data API v3"
3. Click **Enable**

### Step 3: Create OAuth 2.0 Credentials

1. Navigate to **APIs & Services** > **Credentials**
2. Click **Create Credentials** > **OAuth client ID**
3. Select **Web application**
4. Add authorized redirect URI:
   ```
   http://your-hub-domain:${PROACTIONS_HUB_PORT}/youtube/auth/google/callback?target=main
   ```
5. Click **Create**
6. Save the **Client ID** and **Client Secret**

### Step 4: Configure OAuth Consent Screen

1. Navigate to **OAuth consent screen**
2. Select **External** (or Internal for workspace accounts)
3. Fill in required fields:
   - App name
   - User support email
   - Developer contact information
4. Add scopes:
   - `https://www.googleapis.com/auth/youtube.upload`
   - `https://www.googleapis.com/auth/youtube`
5. Add test users (if using External + Testing mode)
6. Save configuration

## YouTube Configuration Structure

```yaml
youtube:
  accounts: array          # Required: List of YouTube accounts
  token_storage_path: string   # Optional: Directory for token files
  token_storage_file: string   # Optional: Token filename pattern
```

### Account Configuration

Each account in the `accounts` array requires:

| Field | Description | Required |
|-------|-------------|----------|
| `id` | Unique identifier for the account | Yes |
| `client_id` | OAuth 2.0 Client ID from Google Cloud Console | Yes |
| `client_secret` | OAuth 2.0 Client Secret from Google Cloud Console | Yes |
| `redirect_uri` | OAuth callback URL (must match Google Cloud Console) | Yes |

### Storage Configuration

| Field | Description | Default |
|-------|-------------|---------|
| `token_storage_path` | Directory path for storing token files | `./tokens` |
| `token_storage_file` | Filename pattern (use `{id}` placeholder) | `youtube_token_{id}.json` |

## Multiple Account Configuration

You can configure multiple YouTube accounts:

```yaml
youtube:
  accounts:
    - id: main
      client_id: ${YOUTUBE_MAIN_CLIENT_ID}
      client_secret: ${YOUTUBE_MAIN_CLIENT_SECRET}
      redirect_uri: ${YOUTUBE_MAIN_REDIRECT_URI}

    - id: secondary
      client_id: ${YOUTUBE_SECONDARY_CLIENT_ID}
      client_secret: ${YOUTUBE_SECONDARY_CLIENT_SECRET}
      redirect_uri: ${YOUTUBE_SECONDARY_REDIRECT_URI}

  token_storage_path: ./tokens
  token_storage_file: youtube_token_{id}.json
```

Each account will have its own token file:
- `./tokens/youtube_token_main.json`
- `./tokens/youtube_token_secondary.json`

## Authentication Flow

### Step 1: Initiate OAuth Flow

Initiate authentication for an account:

```bash
curl http://localhost:${PROACTIONS_HUB_PORT}/youtube/auth/google \
  -H "x-target: main"
```

This returns a redirect URL:
```json
{
  "authUrl": "https://accounts.google.com/o/oauth2/v2/auth?..."
}
```

### Step 2: User Authorization

1. Navigate to the `authUrl` in a browser
2. Sign in to Google account
3. Grant permissions to the application
4. User will be redirected to the callback URL
5. ProActions Hub receives and stores the tokens

### Step 3: Verify Authentication

Check authentication status:

```bash
curl http://localhost:${PROACTIONS_HUB_PORT}/youtube/auth/status \
  -H "x-target: main"
```

Response:
```json
{
  "authenticated": true,
  "target": "main"
}
```

## YouTube API Endpoints

### Authentication Endpoints

| Endpoint | Method | Description | Public |
|----------|--------|-------------|--------|
| `/youtube/auth/google` | GET | Initiate OAuth flow | No |
| `/youtube/auth/google/callback` | GET | OAuth callback handler | Yes |
| `/youtube/auth/status` | GET | Check authentication status | No |
| `/youtube/auth/logout` | POST | Revoke tokens | No |

:::info Public Endpoints
The callback endpoint (`/youtube/auth/google/callback`) is publicly accessible (does not require authentication) to receive OAuth redirects from Google.
:::

### Upload Endpoints

| Endpoint | Method | Description | Public |
|----------|--------|-------------|--------|
| `/youtube/upload` | POST | Upload video with metadata | No |

## Video Upload

### Upload Request Structure

Upload videos using `multipart/form-data`:

```bash
curl -X POST http://localhost:${PROACTIONS_HUB_PORT}/youtube/upload \
  -H "x-target: main" \
  -F "video=@/path/to/video.mp4" \
  -F "title=My Video Title" \
  -F "description=Video description" \
  -F "tags=tag1,tag2,tag3" \
  -F "categoryId=22" \
  -F "privacyStatus=unlisted" \
  -F "thumbnail=@/path/to/thumbnail.jpg"
```

### Required Fields

| Field | Description |
|-------|-------------|
| `video` | Video file (binary) |
| `title` | Video title |

### Optional Fields

| Field | Description | Default |
|-------|-------------|---------|
| `description` | Video description | Empty string |
| `tags` | Comma-separated tags | None |
| `categoryId` | YouTube category ID | `22` (People & Blogs) |
| `privacyStatus` | Privacy status: `public`, `unlisted`, or `private` | `unlisted` |
| `thumbnail` | Custom thumbnail image (JPEG/PNG) | Auto-generated |

### Category IDs

Common YouTube category IDs:

| ID | Category |
|----|----------|
| `1` | Film & Animation |
| `2` | Autos & Vehicles |
| `10` | Music |
| `15` | Pets & Animals |
| `17` | Sports |
| `19` | Travel & Events |
| `20` | Gaming |
| `22` | People & Blogs |
| `23` | Comedy |
| `24` | Entertainment |
| `25` | News & Politics |
| `26` | Howto & Style |
| `27` | Education |
| `28` | Science & Technology |

### Upload Response

Successful upload response:

```json
{
  "success": true,
  "videoId": "dQw4w9WgXcQ",
  "url": "https://www.youtube.com/watch?v=dQw4w9WgXcQ",
  "thumbnailUploaded": true
}
```

## Token Management

### Token Storage

OAuth tokens are stored in JSON files:

```
./tokens/youtube_token_main.json
```

File structure:
```json
{
  "access_token": "ya29.a0...",
  "refresh_token": "1//0g...",
  "scope": "https://www.googleapis.com/auth/youtube.upload",
  "token_type": "Bearer",
  "expiry_date": 1234567890000
}
```

### Automatic Token Refresh

ProActions Hub automatically refreshes expired tokens using the refresh token. No manual intervention is required.

### Token Revocation

Revoke access and delete stored tokens:

```bash
curl -X POST http://localhost:${PROACTIONS_HUB_PORT}/youtube/auth/logout \
  -H "x-target: main"
```

Response:
```json
{
  "success": true,
  "message": "Logged out successfully"
}
```

### Token File Permissions

Ensure the tokens directory is:
- **Writable** by the container user
- **Not world-readable** for security
- **Backed up** regularly (tokens are not easily recoverable)

## Volume Mount for Tokens

### Docker Compose

```yaml
services:
  proactions-hub:
    volumes:
      - ./tokens:/usr/src/app/tokens:rw
```

### Podman

```bash
podman run -v ./tokens:/usr/src/app/tokens:rw proactions-hub
```

:::warning Persistence
Tokens must be stored in a persistent volume. If the volume is lost, users will need to re-authenticate.
:::

## Security Considerations

### OAuth Scopes

Request only necessary scopes:
- `https://www.googleapis.com/auth/youtube.upload` - Upload videos
- `https://www.googleapis.com/auth/youtube` - Full YouTube access

Minimal scope recommendation: Use `youtube.upload` only if you don't need other YouTube API features.

### Redirect URI Validation

Always validate redirect URIs:
1. Use HTTPS in production
2. Include the `target` query parameter in Google Cloud Console configuration
3. Never use wildcards in redirect URIs

### Token Storage Security

Protect token files:
```bash
# Set appropriate permissions
chmod 600 ./tokens/youtube_token_*.json

# Ensure directory is not publicly accessible
chmod 700 ./tokens
```

### Environment Variables

Never commit credentials to version control:
```yaml
# Good
client_secret: ${YOUTUBE_CLIENT_SECRET}

# Bad - never do this
client_secret: your-actual-secret
```

## Logging

YouTube operations are logged when YouTube logging is enabled:

```yaml
logging:
  types:
    youtube:
      enabled: true
      console: true
      file: false
      level: info
```

Log output includes:
- Authentication events
- Upload operations
- Token refresh events
- Error details

See [Logging Configuration](logging.md) for details.

## Testing YouTube Configuration

### Test Authentication Flow

1. **Initiate OAuth:**
   ```bash
   curl http://localhost:${PROACTIONS_HUB_PORT}/youtube/auth/google -H "x-target: main"
   ```

2. **Visit the authUrl** in browser

3. **Complete Google sign-in** and grant permissions

4. **Check authentication status:**
   ```bash
   curl http://localhost:${PROACTIONS_HUB_PORT}/youtube/auth/status -H "x-target: main"
   ```

### Test Video Upload

```bash
curl -X POST http://localhost:${PROACTIONS_HUB_PORT}/youtube/upload \
  -H "x-target: main" \
  -F "video=@test-video.mp4" \
  -F "title=Test Upload" \
  -F "description=Testing ProActions Hub YouTube integration" \
  -F "privacyStatus=unlisted"
```

### Verify Token Storage

Check that token file was created:
```bash
ls -la ./tokens/youtube_token_main.json
```

## Troubleshooting

### Authentication Failed

**Error:** `OAuth authentication failed`

**Solutions:**
1. Verify client ID and secret are correct
2. Check redirect URI matches Google Cloud Console exactly
3. Ensure YouTube Data API v3 is enabled
4. Check OAuth consent screen is properly configured

### Redirect URI Mismatch

**Error:** `redirect_uri_mismatch`

**Solution:**
1. Copy the exact redirect URI from the error message
2. Add it to authorized redirect URIs in Google Cloud Console
3. Include the `target` query parameter

### Token Not Found

**Error:** `No authentication found for target`

**Solutions:**
1. Complete OAuth flow first
2. Check token file exists in tokens directory
3. Verify token storage path configuration
4. Check file permissions

### Token Expired

**Error:** `Token has been expired or revoked`

**Solutions:**
1. ProActions Hub should auto-refresh - check logs
2. If auto-refresh fails, logout and re-authenticate
3. Verify refresh token is present in token file

### Upload Failed

**Error:** Upload returns 4xx or 5xx status

**Solutions:**
1. Verify authentication is valid
2. Check video file format is supported
3. Ensure video file size is within limits
4. Verify required fields are provided

### Permission Denied

**Error:** `Insufficient permissions`

**Solutions:**
1. Check OAuth scopes include necessary permissions
2. Re-authenticate to grant additional scopes
3. Verify account has permission to upload videos

### Quota Exceeded

**Error:** `quotaExceeded`

**Solution:**
YouTube API has daily quota limits. Check quota usage in Google Cloud Console and request increase if needed.

## Best Practices

### Configuration

1. **Use environment variables** for all credentials
2. **Configure multiple accounts** for redundancy
3. **Use descriptive account IDs** for easy identification
4. **Test authentication** after configuration changes

### Security

1. **Use HTTPS** for redirect URIs in production
2. **Protect token files** with appropriate permissions
3. **Backup token files** regularly
4. **Never commit tokens** to version control
5. **Rotate credentials** periodically

### Reliability

1. **Monitor token expiration** through logs
2. **Test refresh flow** regularly
3. **Handle quota limits** gracefully in client applications
4. **Implement upload retries** for transient failures

### Performance

1. **Set appropriate video upload timeouts**
2. **Compress thumbnails** before upload
3. **Use background jobs** for large uploads
4. **Monitor upload success rates**

## Next Steps

- [Configure Tools](tools.md) for content extraction
- [Configure Logging](logging.md) to monitor YouTube operations
- [Review Security Hardening](../operations/security-hardening.md) for production deployments
