---
description: Policy-as-Code and GitOps with WhiteBoxXAI — define drift and bias policy in a whitebox.yaml file, sync it from CI, and evaluate a model against it to warn or block a deployment, with the violations recorded.
---

# Policy-as-Code & GitOps

Keep your organization's model-release policy in the repository, next to the code it governs. A
`whitebox.yaml` file says what should happen when a model has drifted or a bias audit has found bias.
Your CI pipeline syncs it to WhiteBoxXAI and then asks, before a deployment, whether a model passes.
Policy changes go through pull requests and review like any other change, and every evaluation
says which rule caused a warning or a block.

## The policy file

```yaml
# whitebox.yaml
minor_drift_action: warn      # when the model's latest drift report detected drift
major_bias_action: block      # when the model's latest bias audit detected bias
auto_approve_minor: false     # whether a clean evaluation is eligible for automatic approval
```

| Key | Checked against | Values |
| --- | --- | --- |
| `minor_drift_action` | The model's most recent [drift report](../user-guide/assurance.md#drift) | `warn` or `block` |
| `major_bias_action` | The model's most recent [bias audit](../user-guide/assurance.md#fairness) | `warn` or `block` |
| `auto_approve_minor` | | `true` or `false` |

- `block` fails the evaluation (`allowed: false`).
- `warn` lets it pass, with the violation listed.
- A key you leave out of your first sync starts at `warn` for drift and `block` for bias. On later
  syncs, a key you leave out keeps its current value.
- A policy synced before validation was added may still hold `agent_ticket` or `human_review`. Both
  are evaluated as `warn`, so a violation is never silently passed.
- The whole file is stored with the policy, other keys included, so you can see exactly what was
  synced.

## Sync the policy

Send the file's contents to WhiteBoxXAI. Run this on every merge to your main branch:

```bash
curl -X POST https://api.whiteboxxai.com/api/v1/policy/sync \
  -H "Authorization: Bearer $WHITEBOX_API_KEY" \
  -H "Content-Type: application/json" \
  -d "$(python -c 'import json,sys; print(json.dumps({"yaml_content": open("whitebox.yaml").read()}))')"
```

The sync is refused, and the policy in force stays unchanged, when:

- the file isn't valid YAML, or isn't a mapping of keys to values: `400`
- an action is anything other than `warn` or `block` (including a different case, such as `Block`), or
  `auto_approve_minor` isn't `true` or `false`: `422`, naming the key

`GET /api/v1/policy` returns the policy currently in force.

## Evaluate a model

```bash
curl -X POST https://api.whiteboxxai.com/api/v1/policy/evaluate/<model-id> \
  -H "Authorization: Bearer $WHITEBOX_API_KEY"
```

```json
{
  "model_id": "5d1c…",
  "policy_exists": true,
  "allowed": false,
  "action": "block",
  "auto_approve_eligible": false,
  "violations": [
    {"policy_rule": "minor_drift_action", "action": "warn", "detail": "Drift detected (score: 0.31)"},
    {"policy_rule": "major_bias_action", "action": "block", "detail": "Bias detected (overall fairness score: 0.62)"}
  ]
}
```

| Field | Meaning |
| --- | --- |
| `action` | `pass`, `warn` or `block`: the strictest action among the violations |
| `allowed` | `false` only when `action` is `block` |
| `violations` | Each rule that matched, its action, and the evidence behind it |
| `auto_approve_eligible` | `true` when nothing matched and `auto_approve_minor` is on |
| `policy_exists` | `false` if no policy has been synced; the evaluation then passes |

The evaluation reads the model's **latest** drift report and bias audit. Create a fresh drift report
and run a bias audit in the pipeline first if you want the decision based on the build you're about to
deploy. Neither is created by the evaluation itself.

## Gate a deployment in GitHub Actions

The [WhiteBoxXAI Governance Action](https://github.com/AgentaFlow/whitebox-governance-action) runs the
sync and the evaluation in one step, and fails the job on `block`:

```yaml
# .github/workflows/governance.yml
name: Model governance
on:
  push:
    branches: [main]
  pull_request:

jobs:
  policy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - uses: AgentaFlow/whitebox-governance-action@v1
        with:
          api_key: ${{ secrets.WHITEBOX_API_KEY }}
          model_id: ${{ vars.WHITEBOX_MODEL_ID }}
          # Sync only on main, so a pull request can't change the policy in force.
          sync: ${{ github.event_name == 'push' }}
```

On a pull request, the step evaluates the model against the policy already in force. After the merge
to `main`, it syncs the reviewed file first.

| Input | Default | Description |
| --- | --- | --- |
| `api_key` | (required) | A WhiteBoxXAI API key. Store it as a repository secret, never in the workflow file. |
| `policy_file` | `whitebox.yaml` | Path to the policy file. |
| `sync` | `true` | Sync the policy file before evaluating. |
| `model_id` | | The model to evaluate. Leave it empty to sync only. |
| `base_url` | `WHITEBOX_BASE_URL`, then `https://api.whiteboxxai.com` | The API address. On Business Cloud, set it to your tenant's API address. |

| Output | Description |
| --- | --- |
| `action` | `pass`, `warn` or `block` |
| `allowed` | `false` only when `action` is `block` |
| `violations` | The violations as a JSON array |

Each violation appears as an annotation on the run: a warning for `warn`, an error for `block`. The step
also fails on any error from the API, such as a bad key, an unknown model, or a policy file that's
refused. To fail on a warning too, give the step an `id` and check its output:

```yaml
      - uses: AgentaFlow/whitebox-governance-action@v1
        id: policy
        with:
          api_key: ${{ secrets.WHITEBOX_API_KEY }}
          model_id: ${{ vars.WHITEBOX_MODEL_ID }}
      - if: steps.policy.outputs.action == 'warn'
        run: exit 1
```

The Action runs in a Docker container, so it needs a Linux runner. On other runners or CI systems, call
the two endpoints above directly and fail the job when `action` is `block`.

## What it gives an auditor

- The policy is versioned in Git: who changed it, when, and through which reviewed pull request.
- Each evaluation names the rule, the action and the evidence (the drift or bias result) behind a
  warning or block.
- The drift reports and bias audits the evaluation relied on are stored in WhiteBoxXAI with ids you can
  cite, and appear in the [Model Register](../user-guide/model-register.md) dossier.

For approvals that need people rather than a pipeline, route the change through
[Reviews & Decisions](../user-guide/governance.md).
