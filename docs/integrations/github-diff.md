# GitHub Integration Phase 3 - Quick Reference

## Model Version Comparison

### Access Comparison UI

1. Navigate to a model detail page: `/dashboard/models/{model_id}`
2. Click **"Compare Versions"** button (visible when 2+ versions exist)
3. Or visit directly: `/dashboard/models/{model_id}/compare`

### Compare Two Versions

1. **Select Baseline Version** (older version) from dropdown
2. **Select Comparison Version** (newer version) from dropdown
3. View automatic comparison results

---

## API Usage Examples

### Compare Two Model Versions

```bash
curl -X POST "https://api.example.com/v1/github/diff/compare" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "version1_id": 123,
    "version2_id": 124
  }'
```

**Response:**
```json
{
  "has_changes": true,
  "breaking_changes": false,
  "change_summary": "Parameters increased by 1,024 (+5.2%); 2 layer(s) added",
  "parameter_delta": 1024,
  "parameter_change_pct": 5.2,
  "layers_added": ["conv_layer_5", "batch_norm_5"],
  "layers_removed": [],
  "layers_modified": ["dense_output"]
}
```

### Generate Changelog

```bash
curl "https://api.example.com/v1/github/diff/changelog?version1_id=123&version2_id=124" \
  -H "Authorization: Bearer <token>"
```

**Response:**
```json
{
  "changelog": "# Model Changelog: 123 → 124\n\n## Summary\nParameters increased by 1,024 (+5.2%); 2 layer(s) added\n\n## 🟡 Medium Impact Changes\n- **Layer: conv_layer_5** (added)\n- **Hyperparameter: learning_rate** (modified)\n  - `0.001` → `0.0005`"
}
```

### Get Version History

```bash
curl "https://api.example.com/v1/github/diff/model/456/versions?limit=10" \
  -H "Authorization: Bearer <token>"
```

### Get Architecture Timeline

```bash
curl "https://api.example.com/v1/github/diff/architecture-changes/456" \
  -H "Authorization: Bearer <token>"
```

---

## Python SDK Usage

### Compare Models Programmatically

```python
import requests

API_URL = "https://api.example.com"
TOKEN = "your-token"

def compare_versions(version1_id: int, version2_id: int):
    """Compare two model versions."""
    response = requests.post(
        f"{API_URL}/v1/github/diff/compare",
        headers={"Authorization": f"Bearer {TOKEN}"},
        json={"version1_id": version1_id, "version2_id": version2_id}
    )
    return response.json()

# Usage
diff = compare_versions(123, 124)

if diff["breaking_changes"]:
    print("⚠️ Breaking changes detected!")
    print(f"Summary: {diff['change_summary']}")

print(f"Parameter change: {diff['parameter_delta']:+,} ({diff['parameter_change_pct']:+.1f}%)")
print(f"Layers added: {len(diff['layers_added'])}")
print(f"Layers removed: {len(diff['layers_removed'])}")
```

### Generate Release Notes

```python
def generate_release_notes(base_version: int, new_version: int, output_file: str):
    """Generate markdown release notes."""
    response = requests.get(
        f"{API_URL}/v1/github/diff/changelog",
        params={"version1_id": base_version, "version2_id": new_version},
        headers={"Authorization": f"Bearer {TOKEN}"}
    )

    changelog = response.json()["changelog"]

    with open(output_file, "w") as f:
        f.write(changelog)

    print(f"Release notes saved to {output_file}")

# Usage
generate_release_notes(123, 124, "RELEASE_NOTES.md")
```

### Automated Regression Detection

```python
def check_regression(baseline_version: int, new_version: int) -> bool:
    """Check if new version has regressions."""
    diff = compare_versions(baseline_version, new_version)

    # Check for breaking changes
    if diff["breaking_changes"]:
        print("❌ FAIL: Breaking changes detected")
        return False

    # Check parameter increase threshold
    if diff["parameter_change_pct"] > 20:
        print(f"⚠️  WARNING: Parameters increased by {diff['parameter_change_pct']:.1f}%")

    # Check for removed layers
    if diff["layers_removed"]:
        print(f"❌ FAIL: {len(diff['layers_removed'])} layer(s) removed")
        return False

    print("✅ PASS: No regressions detected")
    return True

# Usage in CI/CD
import sys
if not check_regression(baseline_version=123, new_version=124):
    sys.exit(1)  # Fail the build
```

---

## Supported Model Formats

### PyTorch (.pt, .pth)
```python
# Parser extracts:
- Total parameters
- Trainable parameters
- Layer names and configurations
- State dict structure
- Model architecture (Conv, Linear, LSTM, etc.)
```

### TensorFlow (.h5, SavedModel)
```python
# Parser extracts:
- Model type (Sequential, Functional)
- Layer configurations
- Parameter counts
- Optimizer settings
- Input/output shapes
```

### Scikit-learn (.pkl, .joblib)
```python
# Parser extracts:
- Estimator type
- Pipeline steps
- Hyperparameters
- Feature names
- Number of classes
```

### Hugging Face (.safetensors, config.json)
```python
# Parser extracts:
- Model architecture (BERT, GPT, T5, etc.)
- Hidden layers and attention heads
- Vocab size
- Total parameters
- Config from config.json
```

---

## Understanding Comparison Results

### Impact Levels

- **🔴 High Impact**: Breaking changes requiring immediate attention
  - Framework changes
  - Input/output shape modifications
  - Layer removals
  - Architecture changes

- **🟡 Medium Impact**: Significant changes that should be reviewed
  - Layer additions/modifications
  - Hyperparameter changes
  - Parameter changes >10%

- **🟢 Low Impact**: Minor changes with limited effect
  - Small parameter changes <10%
  - Framework version updates

### Change Types

- **Added**: New components (layers, hyperparameters)
- **Removed**: Deleted components (typically breaking)
- **Modified**: Changed components (value or configuration updates)

---

## CI/CD Integration

### GitHub Actions Example

```yaml
name: Model Regression Check

on:
  push:
    paths:
      - 'models/**'

jobs:
  check-regression:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Compare model versions
        run: |
          BASELINE_VERSION=$(git rev-parse HEAD~1)
          NEW_VERSION=$(git rev-parse HEAD)

          curl -X POST "${{ secrets.WhiteBoxXAI_URL }}/v1/github/diff/compare" \
            -H "Authorization: Bearer ${{ secrets.WhiteBoxXAI_TOKEN }}" \
            -H "Content-Type: application/json" \
            -d "{\"version1_id\": \"$BASELINE_VERSION\", \"version2_id\": \"$NEW_VERSION\"}" \
            -o diff.json

      - name: Check for breaking changes
        run: |
          BREAKING=$(jq -r '.breaking_changes' diff.json)
          if [ "$BREAKING" == "true" ]; then
            echo "❌ Breaking changes detected!"
            jq -r '.change_summary' diff.json
            exit 1
          fi
          echo "✅ No breaking changes"
```

### Pre-commit Hook

```bash
#!/bin/bash
# .git/hooks/pre-commit

MODEL_FILES=$(git diff --cached --name-only | grep "models/.*\\.pt$")

if [ -n "$MODEL_FILES" ]; then
    echo "Checking model changes..."

    # Get latest version from WhiteBoxXAI
    LATEST_VERSION=$(curl -s "$WhiteBoxXAI_URL/v1/github/diff/model/$MODEL_ID/versions?limit=1" \
        -H "Authorization: Bearer $WhiteBoxXAI_TOKEN" | jq -r '.[0].id')

    echo "Latest version: $LATEST_VERSION"
    echo "⚠️  Model changes detected. Run comparison manually after push."
fi
```

---

## Troubleshooting

### Comparison Fails

**Problem**: API returns 500 error when comparing versions

**Solutions**:
1. Ensure both versions exist: `GET /v1/github/diff/model/{id}/versions`
2. Check model files are accessible in storage
3. Verify framework dependencies are installed (torch, tensorflow, etc.)
4. Check model file size (<1GB recommended)

### Parser Not Detecting Model

**Problem**: Model file returns "No parser available"

**Solutions**:
1. Check file extension matches supported format
2. Verify file is not corrupted
3. For Hugging Face models, ensure `config.json` exists
4. For TensorFlow, check SavedModel has `saved_model.pb`

### Missing Comparison Data

**Problem**: Frontend shows "No changes detected" but changes exist

**Solutions**:
1. Check both versions use same framework
2. Verify models have complete metadata
3. Try re-parsing models (may need manual intervention)
4. Check parser implementation for your framework

---

## Best Practices

### 1. Regular Comparisons
- Compare every commit to previous version
- Track trends over time
- Set up automated alerts for breaking changes

### 2. Version Selection
- Use tagged versions for production comparisons
- Compare consecutive commits for development
- Maintain baseline versions for regression testing

### 3. Changelog Usage
- Generate changelogs for all releases
- Include in PR descriptions
- Share with stakeholders and team

### 4. Performance Optimization
- Cache comparison results (frontend does this automatically)
- Use version history endpoint efficiently
- Limit timeline queries to recent versions

### 5. Alert Configuration
- Monitor high-impact changes
- Set thresholds for parameter increases
- Alert on breaking changes immediately

---

## Additional Resources

- **Full API Documentation**: `/docs/api/github-diff-api.md`
- **GitHub Configuration Guide**: `/docs/integrations/github-configuration.md`
- **Architecture Documentation**: `/docs/architecture/ARCHITECTURE.md`

---

## Support

For issues or questions:
- GitHub Issues: https://github.com/your-org/whiteboxxai/issues
- Documentation: https://docs.whiteboxxai.com
- Email: support@whiteboxxai.com

---

**Last Updated**: January 2024
**Phase**: 3 - Model Diff & Performance Analysis
**Status**: ✅ Production Ready
