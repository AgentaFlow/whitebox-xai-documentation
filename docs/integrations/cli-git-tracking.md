# CLI Git Tracking Guide

Comprehensive guide for using WhiteBoxXAI's CLI with automatic Git tracking to seamlessly link models to their source code.

## Table of Contents

- [Overview](#overview)
- [Quick Start](#quick-start)
- [Auto-Detection](#auto-detection)
- [Manual Git Metadata](#manual-git-metadata)
- [Examples](#examples)
- [Advanced Usage](#advanced-usage)
- [Troubleshooting](#troubleshooting)

## Overview

WhiteBoxXAI's Python SDK can automatically detect and track Git repository information when registering models. This creates a complete audit trail linking:

- **Models** → **Code commits**
- **Model versions** → **Git branches/tags**
- **Model authors** → **Git commit authors**
- **Model changes** → **Code diffs**

### Benefits

✅ **Automated tracking** - No manual metadata entry
✅ **Complete lineage** - Full model-to-code traceability
✅ **Version control** - Track model changes through Git history
✅ **Collaboration** - See who created/modified models
✅ **Reproducibility** - Know exact code version for any model

## Quick Start

### Installation

```bash
# Install WhiteBoxXAI SDK with Git support
pip install whitebox-xai-sdk

# Git tracking requires GitPython (automatically installed)
# Or install manually:
pip install gitpython
```

### Basic Usage

```python
from whiteboxxai import WhiteBoxXAI

# Initialize client
client = WhiteBoxXAI(
    api_key="your_api_token",
    base_url="https://api.whiteboxxai.com"
)

# Register model with auto Git detection
model = client.models.register(
    name="fraud_detector",
    model_type="classification",
    framework="sklearn",
    auto_detect_git=True  # 🔑 Enable Git auto-detection
)

# Git metadata automatically captured:
# - github_repository_url
# - github_commit_hash
# - github_branch
# - github_commit_author
# - github_commit_message
```

## Auto-Detection

### How It Works

When `auto_detect_git=True`, WhiteBoxXAI:

1. **Searches for .git directory** - Walks up from current directory
2. **Reads repository information** - Extracts remote URL, commit, branch
3. **Captures commit metadata** - Gets author, message, timestamp
4. **Validates information** - Ensures Git context is valid
5. **Submits to platform** - Includes Git data with model registration

### What Gets Captured

| Field | Description | Example |
|-------|-------------|---------|
| `github_repository_url` | Repository URL | `https://github.com/org/repo` |
| `github_commit_hash` | Full commit SHA | `abc123def456...` |
| `github_branch` | Current branch | `main`, `feature/fraud-v2` |
| `github_tag` | Git tag (if on tag) | `v1.0.0` |
| `github_commit_author` | Commit author | `John Doe <john@example.com>` |
| `github_commit_message` | Commit message | `Add fraud detection model` |

### Requirements

**Git Repository**:
- Must be in a Git repository (`.git` directory present)
- Must have at least one commit
- Must have a remote named `origin` (optional but recommended)

**Clean Working Directory** (optional):
- Use `require_clean_git=True` to enforce no uncommitted changes

## Manual Git Metadata

If you prefer manual control, pass Git metadata explicitly:

```python
model = client.models.register(
    name="fraud_detector",
    model_type="classification",
    framework="sklearn",
    # Manual Git metadata
    github_repository_url="https://github.com/myorg/models",
    github_commit_hash="abc123def456",
    github_branch="main",
    github_commit_author="Jane Smith <jane@example.com>",
    github_commit_message="Update fraud model",
    github_tag="v2.0.0"
)
```

**Note**: Manual metadata takes precedence over auto-detected values.

## Examples

### Example 1: Basic Auto-Detection

```python
from whiteboxxai import WhiteBoxXAI

client = WhiteBoxXAI(api_key="wbx_live_...")

# Register with auto-detection
model = client.models.register(
    name="churn_predictor",
    model_type="classification",
    framework="xgboost",
    version="1.0.0",
    auto_detect_git=True
)

print(f"Model registered: {model['id']}")
print(f"Commit: {model['github_commit_hash'][:7]}")
print(f"Branch: {model['github_branch']}")
```

**Output**:
```
Model registered: 42
Commit: abc1234
Branch: main
```

### Example 2: Require Clean Repository

```python
# Fail registration if uncommitted changes exist
try:
    model = client.models.register(
        name="fraud_detector",
        model_type="classification",
        framework="sklearn",
        auto_detect_git=True,
        require_clean_git=True  # 🔒 Enforce clean state
    )
except Exception as e:
    print(f"Registration failed: {e}")
    # Output: "Working directory has uncommitted changes"
```

### Example 3: Combine Auto and Manual

```python
# Auto-detect Git, but override specific fields
model = client.models.register(
    name="sentiment_analyzer",
    model_type="classification",
    framework="pytorch",
    auto_detect_git=True,
    # Override branch (useful in CI/CD)
    github_branch=os.getenv("CI_BRANCH", "main"),
    # Add custom metadata
    deployment_environment="staging"
)
```

### Example 4: Non-Git Environment

```python
# Gracefully handle non-Git environments
model = client.models.register(
    name="recommendation_engine",
    model_type="recommendation",
    framework="tensorflow",
    auto_detect_git=True  # Will log warning but continue
)
# If no Git repo found, model registers without Git metadata
```

### Example 5: Programmatic Git Detection

```python
from whiteboxxai import detect_git_context, validate_git_context

# Manually check Git context first
git_context = detect_git_context()

if git_context:
    print(f"Repository: {git_context.repository_url}")
    print(f"Commit: {git_context.commit_sha[:7]}")
    print(f"Branch: {git_context.branch}")
    print(f"Dirty: {git_context.is_dirty}")

    # Validate before registration
    if validate_git_context(git_context, require_clean=True):
        # Convert to dict for registration
        git_data = git_context.to_dict()

        model = client.models.register(
            name="model_v1",
            model_type="classification",
            framework="sklearn",
            **git_data  # Spread Git metadata
        )
else:
    print("No Git repository found")
```

### Example 6: Async Registration

```python
import asyncio
from whiteboxxai import WhiteBoxXAI

async def register_models():
    client = WhiteBoxXAI(api_key="wbx_live_...")

    # Async registration with Git detection
    model = await client.models.aregister(
        name="async_model",
        model_type="classification",
        framework="sklearn",
        auto_detect_git=True
    )

    return model

# Run async
model = asyncio.run(register_models())
```

## Advanced Usage

### Custom Git Path

```python
from whiteboxxai import detect_git_context

# Detect Git from specific path
git_context = detect_git_context(path="/path/to/repo")

if git_context:
    model = client.models.register(
        name="model",
        model_type="classification",
        framework="sklearn",
        **git_context.to_dict()
    )
```

### SSH vs HTTPS URLs

Git URLs are automatically normalized:

```python
# SSH URL: git@github.com:org/repo.git
# Converted to: https://github.com/org/repo

# Works with both formats automatically
```

### Detached HEAD State

```python
# When in detached HEAD (e.g., checking out a tag)
git_context = detect_git_context()

print(f"Branch: {git_context.branch}")  # None
print(f"Tag: {git_context.tag}")        # v1.0.0

# Branch will be None, but tag is captured
```

### Multiple Remotes

```python
# If multiple remotes exist, 'origin' is used by default
# Override with manual metadata:

model = client.models.register(
    name="model",
    model_type="classification",
    framework="sklearn",
    auto_detect_git=True,
    # Override repository URL
    github_repository_url="https://github.com/different/repo"
)
```

### CI/CD Integration

```python
import os

# Detect Git in CI environment (GitHub Actions example)
github_sha = os.getenv("GITHUB_SHA")
github_ref = os.getenv("GITHUB_REF")
github_repo = os.getenv("GITHUB_REPOSITORY")

if github_sha:  # Running in GitHub Actions
    model = client.models.register(
        name="ci_model",
        model_type="classification",
        framework="sklearn",
        # Use CI environment variables
        github_commit_hash=github_sha,
        github_branch=github_ref.replace("refs/heads/", ""),
        github_repository_url=f"https://github.com/{github_repo}"
    )
else:  # Local development
    model = client.models.register(
        name="ci_model",
        model_type="classification",
        framework="sklearn",
        auto_detect_git=True
    )
```

### Logging Configuration

```python
import logging

# Enable detailed Git detection logging
logging.basicConfig(level=logging.DEBUG)

# Or configure WhiteBoxXAI logger specifically
logger = logging.getLogger("whiteboxxai.git_utils")
logger.setLevel(logging.INFO)

model = client.models.register(
    name="model",
    model_type="classification",
    framework="sklearn",
    auto_detect_git=True
)
# Logs: "Detected Git context: GitContext(repo=..., commit=..., branch=...)"
```

## Troubleshooting

### Issue: GitPython Not Found

**Error**: `GitPython not installed. Install with: pip install gitpython`

**Solution**:
```bash
pip install gitpython
```

WhiteBoxXAI falls back to subprocess-based Git detection if GitPython is unavailable.

### Issue: No Git Repository Found

**Error/Warning**: `No Git repository found at /path`

**Solution**:
- Ensure you're in a Git repository: `git status`
- Check `.git` directory exists
- Run from within repository: `cd /path/to/repo`

### Issue: No Remote URL

**Warning**: `Could not get remote URL`

**Solution**:
```bash
# Add remote
git remote add origin https://github.com/org/repo.git

# Or verify remote exists
git remote -v
```

Models will still register, but `github_repository_url` will be `None`.

### Issue: Uncommitted Changes

**Error**: `Working directory has uncommitted changes (require_clean=True)`

**Solution**:
```bash
# Commit changes
git add .
git commit -m "Your message"

# Or use require_clean_git=False (not recommended for production)
```

### Issue: Detached HEAD

**Warning**: `Could not get branch: detached HEAD`

**Explanation**: You're in detached HEAD state (checking out a specific commit or tag).

**Solution**:
- This is normal for tags/specific commits
- `github_tag` will be captured if on a tag
- `github_branch` will be `None`

### Issue: Permission Denied

**Error**: `PermissionError: [Errno 13] Permission denied: '.git'`

**Solution**:
- Check `.git` directory permissions
- Ensure you have read access to the repository

## Best Practices

### 1. Always Commit Before Registration

```bash
# Workflow
git add models/fraud_detector.pkl
git commit -m "Update fraud detection model"

# Then register
python train.py  # Includes model registration with auto_detect_git=True
```

### 2. Use Tags for Versions

```bash
# Tag release versions
git tag -a v1.0.0 -m "Fraud detector v1.0.0"
git push origin v1.0.0

# Register (tag is auto-detected)
python -c "
from whiteboxxai import WhiteBoxXAI
client = WhiteBoxXAI(api_key='...')
model = client.models.register(
    name='fraud_detector',
    model_type='classification',
    framework='sklearn',
    auto_detect_git=True
)
print(f'Tag: {model[\"github_tag\"]}')  # v1.0.0
"
```

### 3. Require Clean Git in Production

```python
# Development: Allow uncommitted changes
if os.getenv("ENV") == "production":
    require_clean = True
else:
    require_clean = False

model = client.models.register(
    name="model",
    model_type="classification",
    framework="sklearn",
    auto_detect_git=True,
    require_clean_git=require_clean
)
```

### 4. Combine with Model Versioning

```python
import datetime

# Generate semantic version
version = f"1.0.0+{datetime.datetime.now().strftime('%Y%m%d')}"

model = client.models.register(
    name="fraud_detector",
    model_type="classification",
    framework="sklearn",
    version=version,  # 1.0.0+20240119
    auto_detect_git=True
)
```

### 5. Validate Git Context Before Registration

```python
from whiteboxxai import detect_git_context, validate_git_context

# Pre-check
git_context = detect_git_context()

if not git_context:
    print("WARNING: Not in a Git repository!")
    # Decide whether to proceed

if git_context and not validate_git_context(git_context, require_clean=True):
    print("ERROR: Uncommitted changes detected!")
    exit(1)

# Proceed with registration
model = client.models.register(
    name="model",
    model_type="classification",
    framework="sklearn",
    auto_detect_git=True,
    require_clean_git=True
)
```

## Next Steps

- [GitHub Integration](/integrations/github/)
- [SDK Documentation](/sdk/)
- [API Reference](/sdk/api-reference/)
