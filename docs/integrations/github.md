---
description: Connect GitHub to WhiteBoxXAI with OAuth or a personal access token, track repositories, auto-register model versions from commits, trace predictions to code, and add branch protection, check runs and PR comments.
---

# GitHub Integration Guide

## Overview

The GitHub Integration feature allows WhiteBoxXAI to automatically track AI model versions throughout their entire lifecycle—from development in Git to production deployment. By connecting your GitHub repositories, you can:

- **Automatic Version Tracking**: Detect model file changes via webhooks and auto-register new versions
- **Traceability**: Link production predictions back to specific Git commits
- **Version Comparison**: Compare model architectures, parameters, and performance across Git versions
- **CI/CD Integration**: Validate models in GitHub Actions pipelines
- **Audit Trail**: Maintain complete lineage from code to production
- **Branch protection & PR automation**: Require WhiteBoxXAI's model checks before merge, and
  get validation/comparison results posted as PR comments and check runs — see [CI/PR
  Automation](#cipr-automation)

## Table of Contents

1. [Getting Started](#getting-started)
2. [Connection Methods](#connection-methods)
3. [Repository Configuration](#repository-configuration)
4. [Webhook Setup](#webhook-setup)
5. [Configuration File](#configuration-file)
6. [CI/PR Automation](#cipr-automation)
7. [API Reference](#api-reference)
8. [Troubleshooting](#troubleshooting)
9. [Security](#security)

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

These are settings on the tracked repository. What counts as a model file, and how it's registered,
is set in the repository's `.whiteboxxai.yml`; see [Configuration File](#configuration-file).

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

A `.whiteboxxai.yml` file in the root of a tracked repository controls auto-registration. Every key is
optional.

```yaml
# .whiteboxxai.yml
enabled: true                      # auto-register model files from this repository
model_patterns:                    # which files are models (glob patterns)
  - "models/**/*.pkl"
  - "models/**/*.joblib"
exclude_patterns:
  - "**/tests/**"
storage_mode: full                 # "metadata" (default) or "full"
model_name: credit_scoring         # defaults to a name derived from the file
model_type: random_forest          # xgboost, random_forest, neural_network, linear_model, llm, custom
framework: sklearn                 # detected from the file extension if omitted
description: Retail credit approval model
features: [income, utilisation, tenure_months]
target_variable: default_within_12m
baseline_metrics:
  accuracy: 0.94
tags: [production, credit]
```

| Key | Default | Meaning |
| --- | --- | --- |
| `enabled` | `true` | Turn auto-registration on or off for the repository |
| `model_patterns` | `**/*.pkl`, `**/*.h5`, `**/*.pt`, `**/*.onnx`, `**/*.safetensors` | Files treated as models |
| `exclude_patterns` | `**/test/**`, `**/tests/**`, `**/__pycache__/**` | Files never treated as models |
| `storage_mode` | `metadata` | `metadata` records the file's hash, size, commit and parsed architecture. `full` also stores the model file itself, which [server-side explanations](../user-guide/assurance.md#explainability) need. |
| `model_name` | from the file | Name of the Governed AI System the file is registered under |
| `model_type` | `custom` | One of `xgboost`, `random_forest`, `neural_network`, `linear_model`, `llm`, `custom`. Task names such as `classification` aren't model types and fall back to `custom`. |
| `framework` | from the extension | For example `sklearn`, `pytorch`, `tensorflow`, `huggingface` |
| `features`, `target_variable`, `baseline_metrics`, `description`, `tags` | empty | Recorded on the registered model |

Each matching file pushed to a tracked branch registers a new version of the system, with the commit,
branch, author and file hash, so production behaviour traces back to the exact code. With
`storage_mode: full`, the model file is stored too. Files tracked with Git LFS are recorded as
metadata only.

!!! note "Storing model files"
    `storage_mode: full` uploads your model file to WhiteBoxXAI. Use it when you want explanations
    computed by WhiteBoxXAI; leave the default when model files mustn't leave your environment.
    Explanations currently load scikit-learn-style pickle and joblib files.

## CI/PR Automation

Once a repository is tracked, WhiteBoxXAI can require its model checks before a merge, and post
validation and comparison results directly into the pull request — no separate GitHub App to
install. Every call here reuses the OAuth or PAT token already stored on your connection; there's
no new credential to configure.

### Branch protection

Require a WhiteBoxXAI check before a branch can be merged into:

```http
POST /api/v1/github/automation/repositories/{repository_id}/branch-protection/enable
Content-Type: application/json

{
  "branches": ["main"],
  "require_validation": true,
  "require_comparison": true,
  "strict": false
}
```

Inspect or adjust a branch's protection directly:

```http
GET    /api/v1/github/automation/repositories/{repository_id}/branch-protection/{branch}
POST   /api/v1/github/automation/repositories/{repository_id}/branch-protection/{branch}/required-checks
DELETE /api/v1/github/automation/repositories/{repository_id}/branch-protection/{branch}/required-checks?check_context=...
```

### Check runs

Report model validation or comparison results as a GitHub check run, attached to a commit:

```http
POST /api/v1/github/automation/repositories/{repository_id}/checks/validation
POST /api/v1/github/automation/repositories/{repository_id}/checks/comparison
Content-Type: application/json

{
  "head_sha": "abc123...",
  "model_name": "fraud-detection-v3",
  "validation_passed": true,
  "validation_details": { "accuracy": 0.94 },
  "dashboard_url": "https://whiteboxxai.com/dashboard/register?d=..."
}
```

This is what a GitHub Actions workflow calls after running your own validation step, so the
result shows up as a status check on the PR rather than only in your CI logs.

### PR comments

Post the same results as a comment on the pull request itself:

```http
POST /api/v1/github/automation/repositories/{repository_id}/pr-comments/{pr_number}/validation
POST /api/v1/github/automation/repositories/{repository_id}/pr-comments/{pr_number}/comparison
```

Or manage a single running comment — WhiteBoxXAI updates it in place on subsequent calls
instead of posting a new one every time, identified by a hidden marker in the comment body:

```http
PUT /api/v1/github/automation/repositories/{repository_id}/pr-comments/{pr_number}
Content-Type: application/json

{
  "comment_body": "## Model Validation\n\n✅ Accuracy 0.94 (baseline 0.91)",
  "comment_identifier": "<!-- whiteboxxai-bot -->"
}
```

---

## API Reference

### Authentication

All API requests take an [API key](../account/api-keys.md) (or a signed-in session token) in the
`Authorization` header:

```bash
Authorization: Bearer $WHITEBOX_API_KEY
```

Model-diff reviews are included on Business Cloud and Enterprise Edition. See
[GitHub Diff Reviews](github-diff.md).

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
- Disconnect and reconnect GitHub from **Settings → Integrations**
- If you use a GitHub organization with third-party app restrictions, ask an org owner to approve the WhiteBoxXAI app
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
- Check auto-register is enabled on the tracked repository, and `enabled` isn't `false` in `.whiteboxxai.yml`
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
- **In the dashboard**: **Support** in the sidebar footer

---

## What's new

See [What's new](../help/whats-new.md) for recent changes, including model-diff reviews from the model
dossier.
