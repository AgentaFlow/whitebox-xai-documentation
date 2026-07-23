# GitHub Integration Guide

## Overview

The GitHub Integration feature allows WhiteBoxXAI to automatically track AI model versions throughout their entire lifecycle—from development in Git to production deployment. By connecting your GitHub repositories, you can:

- **Automatic Version Tracking**: Detect model file changes via webhooks and auto-register new versions
- **Traceability**: Link production predictions back to specific Git commits
- **Version Comparison**: Compare model architectures, parameters, and performance across Git versions
- **CI/CD Integration**: Validate models in GitHub Actions pipelines
- **Audit Trail**: Maintain complete lineage from code to production

## Table of Contents

1. [Getting Started](#getting-started)
2. [Connection Methods](#connection-methods)
3. [Repository Configuration](#repository-configuration)
4. [Webhook Setup](#webhook-setup)
5. [Configuration File](#configuration-file)
6. [API Reference](#api-reference)
7. [Troubleshooting](#troubleshooting)
8. [Security](#security)

---

## Getting Started

### Prerequisites

- WhiteBoxXAI account with appropriate permissions
- GitHub account with repository access
- For Personal Access Tokens: Ability to create tokens in GitHub settings

### Quick Start

1. **Navigate to Settings**
   - Go to Dashboard → Settings → GitHub Integration

2. **Connect Your Account**
   - Click "Connect with OAuth" (recommended) or "Connect with PAT"
   - Authorize WhiteBoxXAI to access your repositories

3. **Track a Repository**
   - Select a repository from the list
   - Configure tracking options (branches, tags, releases)
   - Enable auto-registration if desired

4. **Set Up Webhooks**
   - Click "Setup Webhook" for your tracked repository
   - WhiteBoxXAI will automatically configure GitHub webhooks

5. **Done!**
   - Model file changes will now be detected automatically
   - New versions will be registered based on your configuration

---

## Connection Methods

WhiteBoxXAI supports two authentication methods for GitHub:

### OAuth (Recommended)

**Pros:**
- Easier setup (no manual token creation)
- Automatic token refresh
- Fine-grained permissions
- Revocable from GitHub settings

**Permissions Required:**
- `repo`: Access to public and private repositories
- `read:org`: Read organization membership

**How to Connect:**
1. Click "Connect with OAuth" in Settings → GitHub Integration
2. Authorize WhiteBoxXAI on GitHub
3. You'll be redirected back with a successful connection

### Personal Access Token (PAT)

**Pros:**
- Works for CI/CD pipelines and automation
- No OAuth callback required
- Suitable for server-to-server authentication

**Cons:**
- Manual token creation required
- No automatic refresh
- Must be manually revoked

**How to Connect:**
1. Create a Personal Access Token at [github.com/settings/tokens/new](https://github.com/settings/tokens/new)
2. Select scopes: `repo` and `read:org`
3. Copy the token (you won't see it again!)
4. In WhiteBoxXAI, click "Connect with PAT"
5. Paste your token and click "Connect"

**Token Scopes Required:**
- `repo`: Full control of private repositories
- `read:org`: Read org and team membership (for organization repos)

---

## Repository Configuration

### Tracking Options

When you track a repository, you can configure:

| Option | Description | Default |
|--------|-------------|---------|
| **Tracked Branches** | List of branches to monitor (e.g., `main`, `production`) | `[]` (all branches) |
| **Track Tags** | Trigger registration when tags are created | `true` |
| **Track Releases** | Trigger registration when releases are published | `true` |
| **Auto-Register** | Automatically register detected models | `false` |
| **Model Path Patterns** | Glob patterns for model files | `**/*.pkl`, `**/*.h5`, `**/*.pt`, `**/*.onnx`, `**/*.safetensors` |

### Example Configuration

```yaml
# .whiteboxxai.yml
version: "1.0"

github:
  auto_register: true
  track_branches: [main, production]
  track_tags: true
  track_releases: true

models:
  - path: "models/production/*.pkl"
    framework: "sklearn"
    model_type: "classification"
    auto_baseline: true

  - path: "models/deep_learning/*.h5"
    framework: "tensorflow"
    model_type: "neural_network"
```

---

## Webhook Setup

### Automatic Setup

WhiteBoxXAI automatically configures webhooks when you click "Setup Webhook":

**Events Subscribed:**
- `push`: Commits to tracked branches
- `release`: Release published
- `create`: Tag created

**Webhook Configuration:**
- **URL**: `https://api.whiteboxxai.com/api/v1/github/webhooks`
- **Content Type**: `application/json`
- **Secret**: Auto-generated for signature verification
- **SSL Verification**: Enabled

### Manual Setup

If automatic setup fails, you can configure webhooks manually:

1. Go to your repository on GitHub
2. Navigate to Settings → Webhooks → Add webhook
3. Configure:
   - **Payload URL**: `https://api.whiteboxxai.com/api/v1/github/webhooks`
   - **Content type**: `application/json`
   - **Secret**: Contact support for your webhook secret
   - **Events**: Select "Let me select individual events"
     - ☑ Pushes
     - ☑ Releases
     - ☑ Branch or tag creation
4. Click "Add webhook"

### Webhook Payload Processing

When GitHub sends a webhook:

1. **Signature Verification**: WhiteBoxXAI verifies the `X-Hub-Signature-256` header
2. **Event Processing**: Determines event type (push, release, create)
3. **File Detection**: Scans for model files matching path patterns
4. **Registration**: Auto-registers models if enabled, or logs for review

---

## Configuration File

The `.whiteboxxai.yml` file in your repository root defines how models are tracked:

### Full Schema

```yaml
version: "1.0"  # Required: Config schema version

github:
  auto_register: true  # Auto-register detected models
  track_branches: [main, staging, production]  # Branches to monitor
  track_tags: true  # Monitor tag creation
  track_releases: true  # Monitor release publication

models:
  - path: "models/**/*.pkl"  # Glob pattern
    framework: "sklearn"  # sklearn, pytorch, tensorflow, huggingface
    model_type: "classification"  # classification, regression, etc.
    features: ["age", "income", "credit_score"]  # Optional
    target_variable: "loan_approved"  # Optional
    auto_baseline: true  # Upload baseline data automatically
    baseline_data_path: "data/baseline.csv"  # Path to baseline data
    tags: ["production", "v2"]  # Model tags

  - path: "ml/deep_learning/*.h5"
    framework: "tensorflow"
    model_type: "neural_network"
    auto_baseline: false

monitoring:
  enable_on_commit: true  # Enable monitoring for new versions
  notification_channels: [slack, email]  # Alert channels

alerts:
  architecture_change: true  # Alert on model architecture changes
  performance_degradation: true  # Alert on performance drop
```

### Minimal Configuration

```yaml
version: "1.0"

models:
  - path: "*.pkl"
    framework: "sklearn"
```

---

## API Reference

### Authentication

All API requests require a JWT token in the `Authorization` header:

```bash
Authorization: Bearer <your-jwt-token>
```

### Endpoints

#### Connect with OAuth

```http
GET /api/v1/github/oauth/authorize
```

Initiates OAuth flow. Redirects to GitHub authorization page.

**Response**: HTTP 302 Redirect

---

#### OAuth Callback

```http
GET /api/v1/github/oauth/callback?code=<code>&state=<state>
```

Handles OAuth callback after user authorization.

**Query Parameters:**
- `code`: Authorization code from GitHub
- `state`: CSRF protection token

**Response**: HTTP 302 Redirect to frontend

---

#### Connect with PAT

```http
POST /api/v1/github/connect/pat
Content-Type: application/json

{
  "personal_access_token": "ghp_xxxxxxxxxxxxxxxxxxxx"
}
```

**Response:**
```json
{
  "id": "uuid",
  "auth_type": "pat",
  "status": "active",
  "github_username": "johndoe",
  "github_email": "john@example.com",
  "scopes": [],
  "created_at": "2026-01-26T10:00:00Z"
}
```

---

#### List Connections

```http
GET /api/v1/github/connections
```

**Response:**
```json
[
  {
    "id": "uuid",
    "auth_type": "oauth",
    "status": "active",
    "github_username": "johndoe",
    "github_email": "john@example.com",
    "scopes": ["repo", "read:org"],
    "created_at": "2026-01-26T10:00:00Z"
  }
]
```

---

#### Verify Connection

```http
POST /api/v1/github/connections/{connection_id}/verify
```

**Response:**
```json
{
  "valid": true
}
```

---

#### List Repositories

```http
GET /api/v1/github/connections/{connection_id}/repositories?page=1&per_page=30
```

**Response:**
```json
{
  "repositories": [
    {
      "id": 12345,
      "name": "ml-models",
      "full_name": "johndoe/ml-models",
      "private": false,
      "default_branch": "main"
    }
  ]
}
```

---

#### Track Repository

```http
POST /api/v1/github/repositories/track
Content-Type: application/json

{
  "connection_id": "uuid",
  "owner": "johndoe",
  "name": "ml-models",
  "tracked_branches": ["main", "production"],
  "tracked_tags": true,
  "tracked_releases": true,
  "auto_register_enabled": true
}
```

**Response:**
```json
{
  "id": "uuid",
  "owner": "johndoe",
  "name": "ml-models",
  "full_name": "johndoe/ml-models",
  "default_branch": "main",
  "is_private": false,
  "tracked_branches": ["main", "production"],
  "webhook_status": "inactive",
  "total_commits_processed": 0,
  "total_models_registered": 0,
  "created_at": "2026-01-26T10:00:00Z"
}
```

---

#### Setup Webhook

```http
POST /api/v1/github/repositories/{repository_id}/webhook
```

**Response:**
```json
{
  "message": "Webhook setup successfully",
  "webhook_id": "123456789",
  "webhook_status": "active"
}
```

---

## Troubleshooting

### Connection Issues

**Problem**: "Invalid Personal Access Token"

**Solution**:
- Ensure token has `repo` and `read:org` scopes
- Check token hasn't expired
- Verify token is for the correct GitHub account

---

**Problem**: OAuth callback fails

**Solution**:
- Check `OAUTH_REDIRECT_URI` is configured correctly in backend settings
- Ensure GitHub OAuth app redirect URI matches configuration
- Clear browser cache and try again

---

### Webhook Issues

**Problem**: Webhook shows "Inactive"

**Solution**:
- Click "Setup Webhook" button
- Verify repository permissions (admin access required)
- Check webhook configuration in GitHub repository settings

---

**Problem**: Models not auto-registering

**Solution**:
- Verify `.whiteboxxai.yml` exists in repository root
- Check `auto_register: true` in configuration
- Ensure model files match path patterns
- Review webhook delivery logs in GitHub repository settings

---

### Permission Issues

**Problem**: "403 Forbidden" when accessing repositories

**Solution**:
- Reconnect GitHub account with correct permissions
- For PAT: Regenerate token with `repo` scope
- For OAuth: Re-authorize with repository access

---

## Security

### Token Encryption

All GitHub access tokens (OAuth and PAT) are encrypted at rest using Fernet symmetric encryption (AES-128-CBC).

### Webhook Signature Verification

All webhook payloads are verified using HMAC-SHA256 signatures before processing. Invalid signatures are rejected.

### Minimal Permissions

WhiteBoxXAI requests only necessary permissions:
- `repo`: Repository access (read-only for most operations)
- `read:org`: Organization membership (for organization repositories)

### Token Revocation

You can revoke WhiteBoxXAI's access anytime:

**OAuth**: Visit [github.com/settings/applications](https://github.com/settings/applications) and revoke WhiteBoxXAI

**PAT**: Delete the connection in WhiteBoxXAI settings, then delete the token in GitHub settings

### Best Practices

1. **Use OAuth for interactive use** (web dashboard)
2. **Use PAT for automation** (CI/CD, scripts)
3. **Rotate PATs regularly** (every 90 days recommended)
4. **Use fine-grained PATs** when available (GitHub feature)
5. **Monitor connection status** and verify periodically
6. **Review webhook logs** for unusual activity

---

## Support

For additional help:
- **Documentation**: [docs.whiteboxxai.com](https://docs.whiteboxxai.com)
- **Email**: support@whiteboxxai.com
- **GitHub Issues**: [github.com/whiteboxxai/issues](https://github.com/whiteboxxai/issues)

---

## Changelog

### Version 1.0 (January 2026)
- Initial GitHub integration release
- OAuth and PAT authentication
- Webhook support for push, release, and tag events
- Auto-registration with `.whiteboxxai.yml` configuration
- Model version tracking and traceability
