# GitHub Integration Quick Reference

## Connection Setup

### OAuth (Recommended)
```
Dashboard → Settings → GitHub Integration → Connect with OAuth
```

### Personal Access Token
```bash
# 1. Create token at github.com/settings/tokens/new
# Required scopes: repo, read:org

# 2. In WhiteBoxXAI:
Dashboard → Settings → GitHub Integration → Connect with PAT
```

## Configuration File

Create `.whiteboxxai.yml` in repository root:

```yaml
version: "1.0"

github:
  auto_register: true
  track_branches: [main, production]
  track_tags: true
  track_releases: true

models:
  - path: "models/**/*.pkl"
    framework: "sklearn"
    model_type: "classification"
    auto_baseline: true
```

## Webhook Events

WhiteBoxXAI processes these GitHub events:
- **push**: Commit to tracked branch → Scan for model files
- **release**: Release published → Register model version
- **create**: Tag created → Link to model version

## CLI Git Tracking

```python
from whiteboxxai import WhiteBoxXAI

client = WhiteBoxXAI(api_key="your-api-key")

# Automatic Git detection (requires GitPython)
client.register_model(
    name="fraud-detector",
    model_type="classification",
    framework="sklearn",
    auto_detect_git=True  # Captures commit hash, branch, repo URL
)
```

## API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/github/oauth/authorize` | GET | Initiate OAuth flow |
| `/github/connect/pat` | POST | Connect with PAT |
| `/github/connections` | GET | List connections |
| `/github/connections/{id}/verify` | POST | Verify connection |
| `/github/repositories/track` | POST | Track repository |
| `/github/repositories/{id}/webhook` | POST | Setup webhook |

## Common Issues

**Q: Models not auto-registering?**
- Check `.whiteboxxai.yml` exists
- Verify `auto_register: true`
- Ensure model files match `path` patterns

**Q: Webhook inactive?**
- Click "Setup Webhook" in UI
- Requires admin access to repository

**Q: Connection fails?**
- OAuth: Check redirect URI configuration
- PAT: Verify `repo` and `read:org` scopes

## Security

- Tokens encrypted at rest (Fernet/AES-128)
- Webhooks verified with HMAC-SHA256
- Minimal permissions requested (repo, read:org)
- OAuth tokens auto-refresh
- Revoke access anytime in GitHub settings

## Model Path Patterns

Default patterns (glob syntax):
```
**/*.pkl        # scikit-learn
**/*.h5         # Keras/TensorFlow
**/*.pt         # PyTorch
**/*.onnx       # ONNX
**/*.safetensors # Hugging Face
```

Override in `.whiteboxxai.yml`:
```yaml
models:
  - path: "production/models/*.pkl"  # Specific folder only
```

## Full Documentation

See the [GitHub Integration guide](/integrations/github/) for the complete walkthrough.
