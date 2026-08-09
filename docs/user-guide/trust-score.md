# Trust Score

The Trust Score is a single 0–100 index per model, combining three signals WhiteBoxXAI
already computes: the model's latest bias/fairness audit, its current drift severity, and
how much of its prediction volume has been explained.

It exists so a non-technical stakeholder — a board, a risk committee, an executive
sponsor — can assess a model at a glance and then drill into what drove the number. The
methodology is documented in full below on purpose: an unexplainable trustworthiness score
would be a strange thing for an explainability platform to ship.

API examples use the base URL `https://api.whiteboxxai.com` and require an
[API key](/account/api-keys/) (`Authorization: Bearer YOUR_API_KEY`).

## How the score is calculated

Three components, each normalized to 0–100, then combined as a weighted average.

### Fairness component

Taken from the model's most recent bias/fairness audit. A model whose latest audit shows no
detected bias scores at the top of this component; higher-severity findings pull it down.

### Drift component

Taken from the model's worst current drift severity:

| Drift severity | Component score |
| --- | --- |
| None | 100 |
| Low | 80 |
| Medium | 55 |
| High | 25 |
| Critical | 0 |

### Explainability component

The ratio of the model's logged predictions that have a generated explanation. A model with
explanations on most of its traffic scores high; one with none scores at the bottom. This is
coverage, not explanation quality.

### Weighting

The three components are combined using your organization's configured weights. The
defaults:

| Component | Default weight |
| --- | --- |
| Fairness | 0.4 |
| Drift | 0.3 |
| Explainability | 0.3 |

**Weights are renormalized over whichever inputs actually exist.** A model with no bias
audit yet still gets a score from drift and explainability alone, with those two weights
scaled back up to sum to 1. This matters when you're onboarding: a new model isn't punished
with a low score for evidence it hasn't had time to produce.

A model with **none** of the three inputs has no score at all — `trust_score` is `null`, and
the model is counted in `models_with_insufficient_data` rather than dragging down your
portfolio average.

!!! note "Scores are recalculated on a schedule"
    Trust Scores are computed by a scheduled background job and stored as snapshots, not
    recalculated on every prediction. A fresh bias audit or drift report will be reflected in
    the score on the next run, which is also what makes the [history
    endpoint](#score-history) meaningful.

## Where to find it

| Surface | What it shows |
| --- | --- |
| **Executive Dashboard** | Portfolio gauge and org-wide average, with lowest-scoring models highlighted. |
| **Model detail page** | That model's score plus the component breakdown. |
| **Trust Score → Settings** | Weight configuration (admins). |
| **Evidence exports** | Trust Score as a category in the compliance evidence package. |

## Get one model's score

```bash
curl https://api.whiteboxxai.com/api/v1/trust-score/{model_id} \
  -H "Authorization: Bearer YOUR_API_KEY"
```

```json
{
  "model_id": "11111111-2222-3333-4444-555555555555",
  "model_name": "fraud-detection-v3",
  "trust_score": 78.5,
  "fairness_component": 85.0,
  "drift_component": 80.0,
  "explainability_component": 66.0,
  "inputs_available": {
    "fairness": true,
    "drift": true,
    "explainability": true
  },
  "last_bias_audit_at": "2026-07-21T09:12:00Z",
  "last_drift_report_at": "2026-07-28T02:00:00Z",
  "explained_predictions_ratio": 0.66,
  "weights": { "fairness": 0.4, "drift": 0.3, "explainability": 0.3 },
  "methodology_note": "Trust Score = weighted average of fairness (bias audit), drift severity, and explainability coverage, renormalized over whichever inputs are available for this model, using this organization's configured (or default) component weights."
}
```

`inputs_available` tells you which of the three signals contributed. Use it to explain a
score to a stakeholder — a 78.5 built from two of three components is a different statement
than one built from all three.

## Portfolio rollup

```bash
GET /api/v1/trust-score/portfolio
```

```json
{
  "org_average_trust_score": 81.2,
  "models": [ { "model_id": "...", "model_name": "...", "trust_score": 78.5, ... } ],
  "models_with_insufficient_data": 2,
  "methodology_note": "..."
}
```

This is what backs the Executive Dashboard gauge. `models_with_insufficient_data` counts
models excluded from the average because none of the three inputs exist yet — worth
surfacing in a board conversation rather than hiding.

## Score history

```bash
GET /api/v1/trust-score/{model_id}/history
```

```json
{
  "model_id": "...",
  "data_points": [
    {
      "snapshot_at": "2026-07-01T00:00:00Z",
      "trust_score": 72.0,
      "fairness_component": 75.0,
      "drift_component": 80.0,
      "explainability_component": 60.0
    }
  ],
  "trend_direction": "improving"
}
```

`trend_direction` is `improving`, `declining`, or `stable`, and is `null` until there are at
least two snapshots. Because each snapshot keeps the component values, you can see *which*
component moved — a declining score driven by drift is a different problem from one driven
by falling explanation coverage.

## Configure the weights

Organization admins can reweight the components to match their own risk priorities:

```bash
GET /api/v1/trust-score/config
```

```json
{
  "fairness_weight": 0.4,
  "drift_weight": 0.3,
  "explainability_weight": 0.3,
  "is_default": true
}
```

`is_default: true` means no override is on file and the defaults above are in use. To
change them:

```bash
PUT /api/v1/trust-score/config
{
  "fairness_weight": 0.5,
  "drift_weight": 0.3,
  "explainability_weight": 0.2
}
```

In the dashboard this lives at **Trust Score → Settings**.

!!! tip "Document your weighting"
    If you reweight, record why. A score is only defensible in an audit if the methodology
    behind it is deliberate and written down — the same reason the default weights are
    published on this page rather than buried in code.

## Not in scope

Two things the Trust Score deliberately doesn't do:

- **Peer benchmarking.** There's no "compare your score to your industry" — no dataset
  exists to support that claim honestly.
- **Automated enforcement.** A low score doesn't pause a model or block a deployment. Acting
  on the score is a human decision, routed through [Governance Review
  Boards](/user-guide/governance/) if you want approval gates.

## Related

- [AI Risk Register](/user-guide/risk-register/) — the structured risk inventory, including entries
  auto-drafted from the same bias and drift signals that feed this score.
- [Auditing for Bias](/user-guide/#auditing-for-bias) — the fairness audits behind the fairness
  component.
- [Detecting Drift](/user-guide/#detecting-drift) — the drift reports behind the drift component.
