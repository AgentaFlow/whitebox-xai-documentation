---
description: Model-diff architecture reviews in WhiteBoxXAI — compare two Git-backed model versions for parameter, layer, hyperparameter, shape and framework changes, generate a changelog, and use the results in CI and governance reviews. Business Cloud and Enterprise Edition.
---

# GitHub Diff Reviews

A reviewer approving a model change needs to know *what actually changed*: not just a new version
number, but whether layers were added, the input shape moved, or the framework was upgraded.
Model-diff reviews compare two versions of a model file from your repository and report those
changes, with an impact level for each, so a review request can cite them.

!!! note "Plan"
    Model-diff architecture reviews are included on **Business Cloud** and **Enterprise Edition**.
    On the Free plan these endpoints return `403`. See [Plans & Limits](../get-started/plans.md).

## Where versions come from

Diffs compare **Git-backed model versions**: model files WhiteBoxXAI has read from a repository you
connected and track. Connect GitHub and track the repository first; see [GitHub](github.md). Each tracked
change to a model file becomes a version you can compare.

List a model's versions to get the ids to compare:

```bash
curl "https://api.whiteboxxai.com/api/v1/github/diff/model/<model-id>/versions?limit=10" \
  -H "Authorization: Bearer $WHITEBOX_API_KEY"
```

Version ids are UUIDs.

## In the dashboard

Open the system in the [Model Register](../user-guide/model-register.md), go to the dossier's **Lineage &
versions** tab, and choose **Compare versions and see the code diff**. Pick a baseline version and a
comparison version.

## Compare two versions

=== "Python SDK"

    ```python
    diff = client.models.compare_versions(version1_id, version2_id)
    print(diff["change_summary"], diff["breaking_changes"])

    changelog = client.models.version_changelog(version1_id, version2_id)   # markdown
    timeline = client.models.architecture_changes_timeline(model_id)
    ```

=== "REST"

    ```bash
    curl -X POST https://api.whiteboxxai.com/api/v1/github/diff/compare \
      -H "Authorization: Bearer $WHITEBOX_API_KEY" -H "Content-Type: application/json" \
      -d '{"version1_id": "<baseline-version-id>", "version2_id": "<new-version-id>"}'

    curl "https://api.whiteboxxai.com/api/v1/github/diff/changelog?version1_id=<id>&version2_id=<id>" \
      -H "Authorization: Bearer $WHITEBOX_API_KEY"

    curl https://api.whiteboxxai.com/api/v1/github/diff/architecture-changes/<model-id> \
      -H "Authorization: Bearer $WHITEBOX_API_KEY"
    ```

=== "MCP"

    `whitebox_models_compare_versions`, `whitebox_models_version_changelog`,
    `whitebox_models_list_git_versions` and `whitebox_models_architecture_timeline`. In clients that
    support MCP Apps, the result renders as a model version diff card. See [MCP Server](mcp.md).

### What a comparison returns

| Field | Meaning |
| --- | --- |
| `has_changes`, `breaking_changes` | Whether anything changed, and whether any change is breaking |
| `change_summary` | One-line summary, for example *"Parameters increased by 1,024 (+5.2%); 2 layer(s) added"* |
| `parameter_delta`, `parameter_change_pct`, `trainable_parameter_delta` | Change in parameter counts |
| `layers_added`, `layers_removed`, `layers_modified` | Layer names, by change type |
| `hyperparameter_changes` | Hyperparameters that changed, old and new |
| `input_shape_changed`, `output_shape_changed` | With old and new shapes |
| `framework_changed`, `framework_version_changed` | With old and new framework names and versions |
| `architecture_changes` | Every individual change, with its type and impact level |

## Supported model formats

| Framework | Files | What's read |
| --- | --- | --- |
| PyTorch | `.pt`, `.pth` | Total and trainable parameters, layer names and configuration, state-dict structure |
| TensorFlow / Keras | `.h5`, SavedModel | Model type, layer configuration, parameter counts, optimizer settings, input and output shapes |
| scikit-learn | `.pkl`, `.joblib` | Estimator type, pipeline steps, hyperparameters, feature names, number of classes |
| Hugging Face | `.safetensors` with `config.json` | Architecture, hidden layers, attention heads, vocabulary size, parameters |

## Impact levels

| Level | Typical changes |
| --- | --- |
| **High** (breaking) | Framework changes, input or output shape changes, removed layers, architecture changes |
| **Medium** | Added or modified layers, hyperparameter changes, parameter changes over 10% |
| **Low** | Parameter changes under 10%, framework version updates |

## Use it in CI

Fail a pull request when a model change is breaking, or post the changelog for reviewers:

```yaml
# .github/workflows/model-diff.yml
name: Model diff
on: pull_request
jobs:
  diff:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/setup-python@v6
        with:
          python-version: "3.12"
      - run: pip install whitebox-xai-sdk
      - name: Compare model versions
        env:
          WHITEBOX_API_KEY: ${{ secrets.WHITEBOX_API_KEY }}
          BASELINE_VERSION_ID: ${{ vars.BASELINE_VERSION_ID }}
          CANDIDATE_VERSION_ID: ${{ vars.CANDIDATE_VERSION_ID }}
        run: |
          python - <<'PY'
          import os, sys
          from whiteboxxai import WhiteBoxXAI
          client = WhiteBoxXAI(api_key=os.environ["WHITEBOX_API_KEY"])
          diff = client.models.compare_versions(os.environ["BASELINE_VERSION_ID"], os.environ["CANDIDATE_VERSION_ID"])
          print(diff["change_summary"])
          sys.exit(1 if diff["breaking_changes"] else 0)
          PY
```

Version ids are UUIDs from the versions list above, not Git commit SHAs.

## Use it in a governance review

Attach the changelog to an **Architecture change** review request in
[Reviews & Decisions](../user-guide/governance.md), so the board votes on the documented change. The
request's evidence snapshot covers what reviewers saw when they voted.

## Troubleshooting

**`403`.** Model diff isn't included in your plan, or your API key's creator lacks access.

**`404 Version not found`.** The version id doesn't exist in your organization. List versions with the
endpoint above.

**No parser for the file.** Check the extension is one of the supported formats; Hugging Face models
need their `config.json`, and TensorFlow SavedModels need `saved_model.pb`.

**"No changes detected" when you expect some.** Check both versions are the same framework and that
the model files were committed in full.
