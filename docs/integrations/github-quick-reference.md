---
description: Quick reference for the WhiteBoxXAI GitHub integration — connecting, tracking repositories, the .whiteboxxai.yml file, SDK Git tracking and the main endpoints.
---

# GitHub Integration Quick Reference

## Connection Setup

### OAuth (Recommended)
```
Settings → Integrations → Manage GitHub connections → Connect with OAuth
```

### Personal Access Token
```bash
# 1. Create token at github.com/settings/tokens/new
# Required scopes: repo, read:org

# 2. In WhiteBoxXAI:
Settings → Integrations → Manage GitHub connections → Connect with PAT
```

## Configuration File

Create `.whiteboxxai.yml` in repository root:

```yaml
enabled: true
model_patterns: ["models/**/*.pkl"]
storage_mode: metadata        # or "full" to store model files (needed for explanations)
model_type: random_forest     # xgboost, random_forest, neural_network, linear_model, llm, custom
framework: sklearn
features: [income, utilisation, tenure_months]
```

See [GitHub → Configuration File](github.md#configuration-file) for every key.

## Webhook Events

WhiteBoxXAI processes these GitHub events:
- **push**: Commit to tracked branch → Scan for model files
- **release**: Release published → Register model version
- **create**: Tag created → Link to model version

## CLI Git Tracking

```python
import os

from whiteboxxai import WhiteBoxXAI

client = WhiteBoxXAI(api_key=os.environ["WHITEBOX_API_KEY"])

# Automatic Git detection (requires GitPython)
client.models.register(
    name="fraud-detector",
    model_type="random_forest",
    framework="sklearn",
    version="1.0.0",
    features=["amount", "merchant_category", "hour"],
    baseline_metrics={},
    auto_detect_git=True,  # captures commit hash, branch and repository URL
)
```

## API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/v1/github/oauth/authorize` | GET | Initiate OAuth flow |
| `/api/v1/github/connect/pat` | POST | Connect with PAT |
| `/api/v1/github/connections` | GET | List connections |
| `/api/v1/github/connections/{id}/verify` | POST | Verify connection |
| `/api/v1/github/repositories/track` | POST | Track repository |
| `/api/v1/github/repositories/{id}/webhook` | POST | Setup webhook |

## Common Issues

**Q: Models not auto-registering?**
- Check `.whiteboxxai.yml` exists
- Verify `enabled: true`
- Ensure model files match `model_patterns`

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
model_patterns: ["production/models/*.pkl"]   # Specific folder only
```

## Full Documentation

See the [GitHub Integration guide](github.md) for the complete walkthrough.
