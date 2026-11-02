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
- Until you set them, the two actions default to `agent_ticket` and `human_review`. With those
  values, a violation is listed but doesn't change the result, so set `warn` or `block` explicitly.
- Keys you leave out of a sync keep their current values. The whole file is stored with the policy,
  so you can see exactly what was synced.

## Sync the policy

Send the file's contents to WhiteBoxXAI. Run this on every merge to your main branch:

```bash
curl -X POST https://api.whiteboxxai.com/api/v1/policy/sync \
  -H "Authorization: Bearer $WHITEBOX_API_KEY" \
  -H "Content-Type: application/json" \
  -d "$(python -c 'import json,sys; print(json.dumps({"yaml_content": open("whitebox.yaml").read()}))')"
```

A file that isn't valid YAML is refused with `400`. `GET /api/v1/policy` returns the policy currently in
force.

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
      - name: Sync policy and evaluate the model
        env:
          WHITEBOX_API_KEY: ${{ secrets.WHITEBOX_API_KEY }}
          MODEL_ID: ${{ vars.WHITEBOX_MODEL_ID }}
        run: |
          python - <<'PY'
          import json, os, sys, urllib.request

          base = "https://api.whiteboxxai.com/api/v1/policy"
          headers = {"Authorization": f"Bearer {os.environ['WHITEBOX_API_KEY']}",
                     "Content-Type": "application/json"}

          def post(url, body=None):
              req = urllib.request.Request(url, data=json.dumps(body or {}).encode(),
                                           headers=headers, method="POST")
              with urllib.request.urlopen(req) as resp:
                  return json.load(resp)

          post(f"{base}/sync", {"yaml_content": open("whitebox.yaml").read()})
          result = post(f"{base}/evaluate/{os.environ['MODEL_ID']}")
          print(f"Policy result: {result['action']}")
          for v in result["violations"]:
              print(f"  [{v['action'].upper()}] {v['policy_rule']}: {v['detail']}")
          sys.exit(1 if result["action"] == "block" else 0)
          PY
```

A `block` fails the job, and the violations are printed in the job log. On Business Cloud, use your
tenant's API address in `base`. Store the API key as a repository secret, never in the workflow file.

## What it gives an auditor

- The policy is versioned in Git: who changed it, when, and through which reviewed pull request.
- Each evaluation names the rule, the action and the evidence (the drift or bias result) behind a
  warning or block.
- The drift reports and bias audits the evaluation relied on are stored in WhiteBoxXAI with ids you can
  cite, and appear in the [Model Register](../user-guide/model-register.md) dossier.

For approvals that need people rather than a pipeline, route the change through
[Reviews & Decisions](../user-guide/governance.md).
