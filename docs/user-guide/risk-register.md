# AI Risk Register

The AI Risk Register is a structured inventory of the risks across your model portfolio.
Each entry has an owner, a likelihood × impact score, a mitigation plan, and a workflow
status — plus a full audit trail of every status change.

This is the artifact auditors ask for by name. ISO/IEC 42001 Clause 6.1 requires
documented AI risk assessment and treatment, and the EU AI Act's risk-management
obligations for high-risk systems expect the same evidence. The register also gives you a
place to map technical controls to the NIST AI Risk Management Framework.

You can manage everything from **Governance & Evidence → Risk Register** in the dashboard,
or through the REST API. API examples below use the base URL
`https://api.whiteboxxai.com` and require an [API key](/account/api-keys/)
(`Authorization: Bearer YOUR_API_KEY`).

## Core concepts

**Risk entry** — a single identified risk, optionally linked to one or more registered
models.

**Risk score** — `likelihood × impact`, where each is scored 1–5. The resulting 1–25 score
maps to a severity band.

**Severity** — `low`, `medium`, `high`, or `critical`, derived from the score using your
organization's thresholds.

**Owner** — the person accountable for the risk. Assigning an owner is what turns a
register from a list into a process.

### Likelihood and impact scales

Both are 1–5. The default labels:

| Value | Likelihood | Impact |
| --- | --- | --- |
| 1 | Rare | Negligible |
| 2 | Unlikely | Minor |
| 3 | Possible | Moderate |
| 4 | Likely | Major |
| 5 | Almost Certain | Severe |

### Severity thresholds

By default, severity is derived from the `likelihood × impact` score:

| Score | Severity |
| --- | --- |
| 20–25 | `critical` |
| 12–19 | `high` |
| 6–11 | `medium` |
| 1–5 | `low` |

Both the scale labels and the thresholds are configurable — see
[Scoring configuration](#scoring-configuration).

### Categories

The default category taxonomy: `bias`, `drift`, `security`, `regulatory`, `operational`,
`reputational`, `third_party`, `other`. You can replace this list with your own taxonomy.

### Workflow states

```text
identified → assessed → mitigation_planned → mitigated → closed
```

Every status change is written to the audit trail with the user who made it, a timestamp,
and a required reason. That history is what makes the register usable as compliance
evidence rather than just a tracker.

## Create a risk entry

In the dashboard, go to **Governance & Evidence → Risk Register → New Risk**, or use the
API:

```bash
curl -X POST https://api.whiteboxxai.com/api/v1/risk-register \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Credit model may under-approve applicants in rural ZIP codes",
    "description": "Fairness audit showed a 9% approval gap by geography.",
    "category": "bias",
    "likelihood": 4,
    "impact": 4,
    "owner_id": "a1b2c3d4-...",
    "mitigation_plan": "Retrain with geographically balanced sampling; re-audit before release.",
    "target_resolution_date": "2026-09-30",
    "review_cadence_days": 30,
    "model_ids": ["11111111-2222-3333-4444-555555555555"]
  }'
```

Returns `201` with the created entry, including the computed `risk_score` (16) and
`severity` (`high`).

**Field reference**

| Field | Notes |
| --- | --- |
| `title` | Required, max 500 characters. |
| `description` | Optional free text. |
| `category` | Required. Must be in your category taxonomy. |
| `likelihood` / `impact` | Required, 1–5 each. |
| `owner_id` | Optional user ID accountable for the risk. |
| `mitigation_plan` | Optional free text. |
| `target_resolution_date` | Optional date. |
| `review_cadence_days` | Optional. Sets `next_review_at` for periodic re-assessment. |
| `model_ids` | Optional list of registered model IDs this risk applies to. |

Read-only fields on the response: `risk_score`, `severity`, `status`, `next_review_at`,
`source_type`, `source_id`, `closed_at`, `created_by`, `created_at`, `updated_at`.

## List and filter entries

```bash
GET /api/v1/risk-register?status=identified&category=bias&skip=0&limit=50
```

The response is paginated:

```json
{
  "items": [ ... ],
  "total": 137,
  "skip": 0,
  "limit": 50
}
```

Filter by category, model, owner, status, and severity. In the dashboard, the same filters
are available above the register table.

## Update an entry

Send only the fields you want to change:

```bash
curl -X PATCH https://api.whiteboxxai.com/api/v1/risk-register/{entry_id} \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "likelihood": 2,
    "mitigation_plan": "Retraining complete; monitoring the approval gap weekly."
  }'
```

Changing `likelihood` or `impact` recomputes `risk_score` and `severity` automatically.

### Moving through the workflow

Status changes go through the same `PATCH`, but **`status_reason` is required** whenever
you include `status`:

```bash
curl -X PATCH https://api.whiteboxxai.com/api/v1/risk-register/{entry_id} \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "status": "mitigated",
    "status_reason": "Retrained model v4.1 closed the approval gap to 1.2%; re-audit passed 2026-08-14."
  }'
```

Omitting `status_reason` on a status change is rejected. The reason becomes the audit-trail
entry — this is deliberate, because "who decided this risk was mitigated, and why" is
exactly what an auditor asks.

## Audit trail

Every create, field edit, and status change is retained:

```bash
GET /api/v1/risk-register/{entry_id}/history
```

```json
{
  "risk_entry_id": "...",
  "events": [
    {
      "event_action": "update",
      "description": "Status changed from mitigation_planned to mitigated",
      "details": { "from": "mitigation_planned", "to": "mitigated" },
      "username": "dana@example.com",
      "created_at": "2026-08-14T16:04:11Z"
    }
  ]
}
```

## Portfolio view and heat map

```bash
GET /api/v1/risk-register/portfolio
```

```json
{
  "heatmap": [ { "likelihood": 4, "impact": 4, "count": 3 }, ... ],
  "top_risks": [ ... ],
  "open_count": 41,
  "open_by_severity": { "critical": 2, "high": 9, "medium": 21, "low": 9 }
}
```

`heatmap` is the 5×5 likelihood × impact grid with a count of open risks per cell.
`top_risks` lists the highest-severity open risks, most severe first. The dashboard renders
this at **Risk Register → Portfolio**.

## Automatic risk drafting

A blank register is the most common way a risk process dies. WhiteBoxXAI drafts entries for
you when the platform detects something that warrants one. Three rules are active:

| Trigger | Creates | Category | Likelihood / Impact |
| --- | --- | --- | --- |
| A bias audit detects bias at `high` or `critical` severity | "Bias/fairness threshold breach — *model*" | `bias` | 4 or 5 / 4 |
| A drift report lands at `high` or `critical` severity | "High-severity drift detected — *model*" | `drift` | 4 or 5 / 3 |
| A governance review request is **denied** or sent back with **request changes** | "Governance review flagged: *title*" | `operational` | 3 / 3 |

Auto-drafted entries arrive in `identified` status with `source_type` set to `bias_audit`,
`drift_report`, or `governance_review`, and `source_id` pointing at the originating record.
They are ordinary entries from there on — assign an owner, adjust the scoring, and work them
through the workflow.

!!! note "No duplicates"
    If an open entry already exists for the same source record, a repeat trigger is skipped
    rather than creating a second entry. A flapping drift detector won't flood your register.

## Scoring configuration

Organization admins can replace the default scales, taxonomy, and thresholds:

```bash
GET /api/v1/risk-register/config
```

```json
{
  "likelihood_scale": [ { "value": 1, "label": "Rare" }, ... ],
  "impact_scale": [ { "value": 1, "label": "Negligible" }, ... ],
  "category_taxonomy": ["bias", "drift", "security", ...],
  "severity_thresholds": { "critical": 20, "high": 12, "medium": 6 },
  "is_default": true
}
```

`is_default: true` means your organization has no override on file and is using the
defaults documented above. To change them:

```bash
PUT /api/v1/risk-register/config
{
  "category_taxonomy": ["bias", "drift", "security", "regulatory", "model_risk"],
  "severity_thresholds": { "critical": 16, "high": 10, "medium": 5 }
}
```

Send only the parts you want to override. In the dashboard this lives at
**Risk Register → Settings**.

!!! warning "Changing thresholds is not retroactive"
    New thresholds apply when an entry's `likelihood` or `impact` next changes. Existing
    entries keep their stored `severity` until they're edited.

## Evidence exports

Risk register data is available as an export category, so the register can be included in
the compliance evidence package you hand to an auditor alongside bias audits, drift
reports, and governance decisions. See [Evidence & Reports](/user-guide/#evidence--reports).

## Related

- [Trust Score](/user-guide/trust-score/) — the aggregate 0–100 index over fairness, drift, and
  explainability signals.
- [Governance Review Boards](/user-guide/governance/) — multi-party approval workflows, one of the
  three sources that auto-draft risk entries.
- [AI Regulations](/account/ai-regulations/) — background on the wider regulatory
  landscape.

!!! info "Two different risk views"
    The Executive Dashboard also shows a computed, read-only risk heat map derived from
    alert data (`GET /api/v1/executive-dashboard/risk-register`). That is a live snapshot,
    not this register — it has no owners, workflow, or history. The AI Risk Register
    described on this page is the persisted GRC artifact.
