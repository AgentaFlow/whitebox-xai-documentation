# WhiteBoxXAI API Reference

Complete API documentation for the WhiteBoxXAI platform.

**Base URL:** `https://api.whiteboxxai.com/api/v1`

---

## Table of Contents

1. [Authentication](#authentication)
2. [Models](#models)
3. [Predictions](#predictions)
4. [Metrics](#metrics)
5. [Drift Detection](#drift-detection)
6. [Explainability (XAI)](#explainability-xai)
7. [Bias & Fairness](#bias--fairness)
8. [LLM Monitoring](#llm-monitoring)
9. [Trust Score](#trust-score)
10. [Risk Register](#risk-register)
11. [Governance Review Boards](#governance-review-boards)
12. [Alerts](#alerts)
13. [Exports & Reports](#exports--reports)
14. [Users & RBAC](#users--rbac)
15. [Dashboard](#dashboard)
16. [Error Handling](#error-handling)

---

## Authentication

### Overview

Every request (except login and register) needs a bearer token in the `Authorization`
header. Two kinds of token are accepted, and they're sent identically:

```http
GET /models
Authorization: Bearer <token>
```

| Token | Lifetime | Use for |
| --- | --- | --- |
| **API key** (`wbx_live_...`) | Until revoked, or its optional expiry | The SDK, CI/CD, pipelines, the MCP server, webhooks |
| **Login JWT** | ~30 minutes | Interactive scripts and the dashboard |

For anything running unattended, use an API key. A JWT expires after about 30 minutes and
carries the full privileges of a human account, which is the wrong credential for a nightly
job.

### API keys

Creation and revocation are admin-only, since a key is an organization-wide credential.

```http
POST /api-keys
Content-Type: application/json

{
  "name": "CI pipeline",
  "scopes": ["read", "write"],
  "expires_in_days": 365
}
```

The `201` response contains `raw_key` — the **only** time the key material is available.
Store it immediately.

```http
GET    /api-keys            # List keys (metadata only, never key material)
DELETE /api-keys/{key_id}   # Revoke a key -> 204, effective immediately
```

See [API Keys](/account/api-keys/) for the full reference.

### Login
```http
POST /auth/login
Content-Type: application/json

{
  "username": "user@example.com",
  "password": "your_password"
}
```

**Response:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer",
  "expires_in": 1800,
  "user": {
    "id": "uuid",
    "username": "user@example.com",
    "role": "admin"
  }
}
```

### Register
```http
POST /auth/register
Content-Type: application/json

{
  "username": "user@example.com",
  "email": "user@example.com",
  "password": "secure_password",
  "full_name": "John Doe"
}
```

---

## Models

### Register Model
Register a new ML model for monitoring.

```http
POST /models
Authorization: Bearer {token}
Content-Type: application/json

{
  "name": "fraud_detection_v1",
  "description": "Credit card fraud detection model",
  "model_type": "classification",
  "framework": "scikit-learn",
  "version": "1.0.0",
  "metadata": {
    "algorithm": "random_forest",
    "features": ["amount", "merchant_category", "time_of_day"],
    "target": "is_fraud",
    "training_samples": 100000
  },
  "baseline_metrics": {
    "accuracy": 0.95,
    "precision": 0.92,
    "recall": 0.88,
    "f1_score": 0.90
  }
}
```

**Response (201 Created):**
```json
{
  "id": "model_uuid",
  "name": "fraud_detection_v1",
  "model_type": "classification",
  "framework": "scikit-learn",
  "version": "1.0.0",
  "status": "active",
  "created_at": "2025-12-05T10:00:00Z",
  "monitoring_enabled": true
}
```

### List Models
```http
GET /models?status_filter=ACTIVE&model_type=classification&skip=0&limit=100
Authorization: Bearer {token}
```

**Query Parameters:**
- `status_filter` (optional): Filter by status (`ACTIVE`, `INACTIVE`, `DEPRECATED`, `ARCHIVED`)
- `model_type` (optional): Filter by type (classification, regression, clustering, llm)
- `owner_id` (optional): Filter by owner UUID
- `tags` (optional): Comma-separated list of tags
- `search` (optional): Search term matched against name and description
- `skip` (optional): Number of results to skip (default: 0)
- `limit` (optional): Items per page (default: 100, max: 1000)

**Response (200 OK):** a JSON array of model objects.
```json
[
  {
    "id": "model_uuid",
    "name": "fraud_detection_v1",
    "model_type": "classification",
    "framework": "scikit-learn",
    "version": "1.0.0",
    "status": "ACTIVE",
    "created_at": "2025-12-05T10:00:00Z"
  }
]
```

### Get Model Details
```http
GET /models/{model_id}
Authorization: Bearer {token}
```

**Response (200 OK):**
```json
{
  "id": "model_uuid",
  "name": "fraud_detection_v1",
  "description": "Credit card fraud detection model",
  "model_type": "classification",
  "framework": "scikit-learn",
  "version": "1.0.0",
  "status": "active",
  "metadata": {
    "algorithm": "random_forest",
    "features": ["amount", "merchant_category", "time_of_day"]
  },
  "baseline_metrics": {
    "accuracy": 0.95,
    "precision": 0.92
  },
  "monitoring_config": {
    "sampling_rate": 0.1,
    "drift_threshold": 0.05
  },
  "created_at": "2025-12-05T10:00:00Z",
  "updated_at": "2025-12-05T15:30:00Z"
}
```

### Update Model
```http
PATCH /models/{model_id}
Authorization: Bearer {token}
Content-Type: application/json

{
  "description": "Updated description"
}
```

### Update Model Status
```http
PATCH /models/{model_id}/status
Authorization: Bearer {token}
Content-Type: application/json

{
  "status": "DEPRECATED"
}
```

### Update Baseline
Refresh a model's baseline metrics, e.g. after retraining.

```http
PATCH /models/{model_id}/baseline
Authorization: Bearer {token}
```

### Versions
```http
GET /models/{model_name}/versions   # All versions, newest first
GET /models/{model_name}/latest     # Most recent version
Authorization: Bearer {token}
```

### Archive and Restore Model
```http
POST /models/{model_id}/archive
POST /models/{model_id}/restore
Authorization: Bearer {token}
```

### Delete Model
Permanent. Prefer archiving.

```http
DELETE /models/{model_id}
Authorization: Bearer {token}
```

Returns `204`.

---

## Predictions

### Log Prediction
Log a single prediction for monitoring.

```http
POST /predictions/log
Authorization: Bearer {token}
Content-Type: application/json

{
  "model_id": "model_uuid",
  "prediction_id": "pred_12345",
  "input_data": {
    "amount": 150.50,
    "merchant_category": "grocery",
    "time_of_day": "afternoon"
  },
  "output_data": {
    "prediction": 0,
    "probability": 0.92,
    "class": "not_fraud"
  },
  "latency_ms": 12.4,
  "metadata": {
    "user_id": "user_789",
    "request_id": "req_xyz"
  }
}
```

!!! warning "Field names are `input_data` and `output_data`"
    Not `inputs`/`output`. A request using the wrong names is rejected as a validation error.

**Response (201 Created):**
```json
{
  "model_id": "model_uuid",
  "prediction_id": "pred_12345",
  "input_data": { "amount": 150.50 },
  "output_data": { "prediction": 0, "probability": 0.92 },
  "latency_ms": 12.4,
  "timestamp": "2025-12-05T15:30:00Z"
}
```

### Batch Log Predictions
Up to 1,000 predictions per call.

```http
POST /predictions/log/batch
Authorization: Bearer {token}
Content-Type: application/json

{
  "model_id": "model_uuid",
  "predictions": [
    {
      "model_id": "model_uuid",
      "prediction_id": "pred_001",
      "input_data": {...},
      "output_data": {...}
    },
    {
      "model_id": "model_uuid",
      "prediction_id": "pred_002",
      "input_data": {...},
      "output_data": {...}
    }
  ]
}
```

### Get a Prediction
```http
GET /predictions/{prediction_id}
Authorization: Bearer {token}
```

### Query Predictions
Filtering is a `POST` with a JSON body, not query-string parameters.

```http
POST /predictions/query
Authorization: Bearer {token}
Content-Type: application/json

{
  "model_id": "model_uuid",
  "start_time": "2025-12-01T00:00:00Z",
  "end_time": "2025-12-05T00:00:00Z",
  "limit": 100,
  "offset": 0
}
```

**Body Parameters:**

- `model_id` (optional): Model UUID
- `start_time` / `end_time` (optional): ISO 8601 timestamps
- `limit` (optional): Max results (default: 100, max: 1000)
- `offset` (optional): Results to skip (default: 0)

**Response (200 OK):** a JSON array of prediction objects.

### Prediction Statistics
```http
GET /predictions/models/{model_id}/stats?start_time=2025-12-01T00:00:00Z&end_time=2025-12-05T00:00:00Z
Authorization: Bearer {token}
```

Returns `total_predictions`, the time period, and latency statistics
(`avg_latency_ms`, `min_latency_ms`, `max_latency_ms`, `predictions_per_hour`).

### Recent Predictions
```http
GET /predictions/models/{model_id}/recent?limit=10
Authorization: Bearer {token}
```

`limit` defaults to 10, max 100.

---

## Metrics

### Log Metrics
```http
POST /metrics
Authorization: Bearer {token}
Content-Type: application/json

{
  "model_id": "model_uuid",
  "metric_type": "accuracy",
  "value": 0.94,
  "metadata": {
    "dataset_size": 1000,
    "evaluation_date": "2025-12-05"
  }
}
```

### Batch Log Metrics
```http
POST /metrics/batch
Authorization: Bearer {token}
Content-Type: application/json

{
  "model_id": "model_uuid",
  "metrics": [
    {"metric_type": "accuracy", "value": 0.94},
    {"metric_type": "precision", "value": 0.91},
    {"metric_type": "recall", "value": 0.89},
    {"metric_type": "f1_score", "value": 0.90}
  ]
}
```

### Calculate Metrics
Calculate metrics from predictions.

```http
POST /metrics/calculate
Authorization: Bearer {token}
Content-Type: application/json

{
  "model_id": "model_uuid",
  "y_true": [0, 1, 1, 0, 1],
  "y_pred": [0, 1, 0, 0, 1],
  "metric_types": ["accuracy", "precision", "recall", "f1_score"]
}
```

**Response (200 OK):**
```json
{
  "model_id": "model_uuid",
  "metrics": {
    "accuracy": 0.8,
    "precision": 0.67,
    "recall": 0.67,
    "f1_score": 0.67
  },
  "calculated_at": "2025-12-05T15:30:00Z"
}
```

### Get Aggregated Metrics
```http
GET /metrics/aggregate?model_id={model_id}&metric_type=accuracy&aggregation=avg&time_window=7d
Authorization: Bearer {token}
```

**Query Parameters:**
- `model_id` (required): Model UUID
- `metric_type` (required): Metric type
- `aggregation` (required): Aggregation function (avg, min, max, sum, count)
- `time_window` (required): Time window (1h, 24h, 7d, 30d)
- `start_date` (optional): Start date
- `end_date` (optional): End date

**Response (200 OK):**
```json
{
  "model_id": "model_uuid",
  "metric_type": "accuracy",
  "aggregation": "avg",
  "time_window": "7d",
  "value": 0.93,
  "data_points": 168,
  "period": {
    "start": "2025-11-28T00:00:00Z",
    "end": "2025-12-05T00:00:00Z"
  }
}
```

### Get Metrics Trend
```http
GET /metrics/trend?model_id={model_id}&metric_type=accuracy&interval=1d&days=30
Authorization: Bearer {token}
```

**Response (200 OK):**
```json
{
  "model_id": "model_uuid",
  "metric_type": "accuracy",
  "interval": "1d",
  "data": [
    {"timestamp": "2025-11-05T00:00:00Z", "value": 0.95},
    {"timestamp": "2025-11-06T00:00:00Z", "value": 0.94},
    {"timestamp": "2025-11-07T00:00:00Z", "value": 0.93}
  ]
}
```

---

## Drift Detection

### Detect Data Drift
```http
POST /drift/detect
Authorization: Bearer {token}
Content-Type: application/json

{
  "model_id": "model_uuid",
  "reference_data": {
    "feature1": [1.2, 3.4, 2.1, ...],
    "feature2": ["A", "B", "A", ...]
  },
  "current_data": {
    "feature1": [1.5, 3.8, 2.5, ...],
    "feature2": ["A", "C", "B", ...]
  },
  "config": {
    "threshold": 0.05,
    "method": "ks_test"
  }
}
```

**Response (200 OK):**
```json
{
  "model_id": "model_uuid",
  "drift_detected": true,
  "overall_drift_score": 0.12,
  "features": {
    "feature1": {
      "drift_detected": true,
      "drift_score": 0.08,
      "p_value": 0.001,
      "method": "ks_test",
      "severity": "medium"
    },
    "feature2": {
      "drift_detected": true,
      "drift_score": 0.15,
      "p_value": 0.0001,
      "method": "chi_squared",
      "severity": "high"
    }
  },
  "timestamp": "2025-12-05T15:30:00Z"
}
```

### Get Drift Reports
```http
GET /drift/reports?model_id={model_id}&drift_type=data&start_date=2025-11-01
Authorization: Bearer {token}
```

**Query Parameters:**
- `model_id` (required): Model UUID
- `drift_type` (optional): Filter by type (data, concept)
- `severity` (optional): Filter by severity (low, medium, high)
- `start_date` (optional): Start date
- `end_date` (optional): End date

### Get Drift Trend
```http
GET /drift/trend?model_id={model_id}&feature=feature1&days=30
Authorization: Bearer {token}
```

---

## Explainability (XAI)

### Request Explanation
Generate an explanation for a prediction.

```http
POST /explanations/generate
Authorization: Bearer {token}
Content-Type: application/json

{
  "model_id": "model_uuid",
  "prediction_id": "pred_12345",
  "method": "shap",
  "config": {
    "background_samples": 100,
    "max_features": 10
  }
}
```

**Response (202 Accepted):**
```json
{
  "explanation_id": "exp_uuid",
  "status": "processing",
  "estimated_time": 5
}
```

### Get Explanation
```http
GET /explanations/{explanation_id}
Authorization: Bearer {token}
```

**Response (200 OK):**
```json
{
  "id": "exp_uuid",
  "model_id": "model_uuid",
  "prediction_id": "pred_12345",
  "method": "shap",
  "status": "completed",
  "result": {
    "base_value": 0.2,
    "predicted_value": 0.92,
    "feature_contributions": {
      "amount": 0.15,
      "merchant_category": -0.05,
      "time_of_day": 0.62
    },
    "top_features": [
      {"feature": "time_of_day", "value": 0.62, "impact": "positive"},
      {"feature": "amount", "value": 0.15, "impact": "positive"}
    ]
  },
  "created_at": "2025-12-05T15:30:00Z",
  "completed_at": "2025-12-05T15:30:05Z"
}
```

### Get Global Feature Importance
```http
GET /explanations/global?model_id={model_id}&method=shap
Authorization: Bearer {token}
```

**Response (200 OK):**
```json
{
  "model_id": "model_uuid",
  "method": "shap",
  "feature_importance": {
    "time_of_day": 0.45,
    "amount": 0.30,
    "merchant_category": 0.15,
    "location": 0.10
  },
  "calculated_at": "2025-12-05T15:30:00Z"
}
```

---

## Bias & Fairness

### Run Fairness Audit
```http
POST /bias/audit
Authorization: Bearer {token}
Content-Type: application/json

{
  "model_id": "model_uuid",
  "protected_attributes": ["gender", "age_group"],
  "reference_group": {"gender": "male", "age_group": "30-50"},
  "metrics": ["demographic_parity", "equal_opportunity", "equalized_odds"],
  "threshold": 0.8
}
```

**Response (202 Accepted):**
```json
{
  "audit_id": "audit_uuid",
  "status": "processing"
}
```

### Get Audit Results
```http
GET /bias/audits/{audit_id}
Authorization: Bearer {token}
```

**Response (200 OK):**
```json
{
  "id": "audit_uuid",
  "model_id": "model_uuid",
  "status": "completed",
  "overall_fairness_score": 0.85,
  "bias_detected": true,
  "protected_attributes": ["gender", "age_group"],
  "results": {
    "demographic_parity": {
      "passed": false,
      "score": 0.72,
      "threshold": 0.8,
      "groups": {
        "male_30-50": {"rate": 0.45},
        "female_30-50": {"rate": 0.62},
        "male_50+": {"rate": 0.40}
      }
    },
    "equal_opportunity": {
      "passed": true,
      "score": 0.88
    }
  },
  "recommendations": [
    "Consider rebalancing training data for gender groups",
    "Review feature importance for age-related features"
  ],
  "created_at": "2025-12-05T15:30:00Z"
}
```

### List Audits
```http
GET /bias/audits?model_id={model_id}&status=completed
Authorization: Bearer {token}
```

---

## LLM Monitoring

### Log LLM Completion
```http
POST /llm/completions
Authorization: Bearer {token}
Content-Type: application/json

{
  "model_id": "model_uuid",
  "provider": "openai",
  "model_name": "gpt-4",
  "prompt": "Explain quantum computing",
  "completion": "Quantum computing is...",
  "tokens": {
    "prompt_tokens": 15,
    "completion_tokens": 120,
    "total_tokens": 135
  },
  "latency_ms": 1500,
  "cost": 0.0027,
  "metadata": {
    "temperature": 0.7,
    "max_tokens": 200
  }
}
```

### Get LLM Metrics
```http
GET /llm/metrics?model_id={model_id}&start_date=2025-12-01&end_date=2025-12-05
Authorization: Bearer {token}
```

**Response (200 OK):**
```json
{
  "model_id": "model_uuid",
  "period": {
    "start": "2025-12-01T00:00:00Z",
    "end": "2025-12-05T23:59:59Z"
  },
  "totals": {
    "completions": 15420,
    "total_tokens": 2345678,
    "prompt_tokens": 1234567,
    "completion_tokens": 1111111,
    "total_cost": 47.89,
    "avg_latency_ms": 1245
  },
  "by_model": {
    "gpt-4": {
      "completions": 10000,
      "cost": 35.00
    },
    "gpt-3.5-turbo": {
      "completions": 5420,
      "cost": 12.89
    }
  }
}
```

### Analyze Toxicity
```http
POST /llm/toxicity
Authorization: Bearer {token}
Content-Type: application/json

{
  "text": "Example text to analyze",
  "completion_id": "completion_uuid"
}
```

**Response (200 OK):**
```json
{
  "completion_id": "completion_uuid",
  "toxicity_detected": false,
  "scores": {
    "toxicity": 0.02,
    "severe_toxicity": 0.001,
    "obscene": 0.01,
    "threat": 0.002,
    "insult": 0.015,
    "identity_attack": 0.003
  },
  "flagged": false
}
```

---

## Trust Score

A 0–100 index per model over fairness, drift, and explainability signals. See [Trust
Score](/user-guide/trust-score/) for the methodology.

### Get a Model's Trust Score
```http
GET /trust-score/{model_id}
Authorization: Bearer {token}
```

**Response (200 OK):**
```json
{
  "model_id": "model_uuid",
  "model_name": "fraud-detection-v3",
  "trust_score": 78.5,
  "fairness_component": 85.0,
  "drift_component": 80.0,
  "explainability_component": 66.0,
  "inputs_available": { "fairness": true, "drift": true, "explainability": true },
  "last_bias_audit_at": "2026-07-21T09:12:00Z",
  "last_drift_report_at": "2026-07-28T02:00:00Z",
  "explained_predictions_ratio": 0.66,
  "weights": { "fairness": 0.4, "drift": 0.3, "explainability": 0.3 },
  "methodology_note": "..."
}
```

`trust_score` is `null` when none of the three inputs exist for the model. Weights are
renormalized over whichever inputs are available.

### Portfolio Rollup
```http
GET /trust-score/portfolio
Authorization: Bearer {token}
```

**Response (200 OK):**
```json
{
  "org_average_trust_score": 81.2,
  "models": [ { "model_id": "...", "model_name": "...", "trust_score": 78.5 } ],
  "models_with_insufficient_data": 2,
  "methodology_note": "..."
}
```

### Score History
```http
GET /trust-score/{model_id}/history
Authorization: Bearer {token}
```

Returns `data_points` (each with `snapshot_at` and the component values) and
`trend_direction` — `improving`, `declining`, `stable`, or `null` with fewer than two points.

### Weight Configuration
```http
GET /trust-score/config
PUT /trust-score/config          # org admin only
Authorization: Bearer {token}
Content-Type: application/json

{
  "fairness_weight": 0.5,
  "drift_weight": 0.3,
  "explainability_weight": 0.2
}
```

`GET` returns the weights in use plus `is_default`, which is `true` when no override is on
file.

---

## Risk Register

Structured AI risk inventory with owners, scoring, workflow, and audit trail. See [AI Risk
Register](/user-guide/risk-register/).

### Create a Risk Entry
```http
POST /risk-register
Authorization: Bearer {token}
Content-Type: application/json

{
  "title": "Credit model may under-approve rural applicants",
  "description": "Fairness audit showed a 9% approval gap by geography.",
  "category": "bias",
  "likelihood": 4,
  "impact": 4,
  "owner_id": "user_uuid",
  "mitigation_plan": "Retrain with balanced sampling; re-audit before release.",
  "target_resolution_date": "2026-09-30",
  "review_cadence_days": 30,
  "model_ids": ["model_uuid"]
}
```

**Response (201 Created):** the entry with computed `risk_score` (16) and `severity`
(`high`), plus `status` (`identified`) and `next_review_at`.

`likelihood` and `impact` are 1–5. `category` must be in your organization's taxonomy.

### List Risk Entries
```http
GET /risk-register?status=identified&category=bias&skip=0&limit=50
Authorization: Bearer {token}
```

**Response (200 OK):**
```json
{ "items": [ ... ], "total": 137, "skip": 0, "limit": 50 }
```

### Get / Update a Risk Entry
```http
GET   /risk-register/{entry_id}
PATCH /risk-register/{entry_id}
Authorization: Bearer {token}
Content-Type: application/json

{
  "status": "mitigated",
  "status_reason": "Retrained v4.1 closed the gap to 1.2%; re-audit passed."
}
```

`status_reason` is **required** whenever `status` is included — it becomes the audit-trail
entry. Changing `likelihood` or `impact` recomputes `risk_score` and `severity`.

Valid statuses: `identified`, `assessed`, `mitigation_planned`, `mitigated`, `closed`.

### Entry History
```http
GET /risk-register/{entry_id}/history
Authorization: Bearer {token}
```

Returns every create, edit, and status change with the acting user and timestamp.

### Portfolio Heat Map
```http
GET /risk-register/portfolio
Authorization: Bearer {token}
```

**Response (200 OK):**
```json
{
  "heatmap": [ { "likelihood": 4, "impact": 4, "count": 3 } ],
  "top_risks": [ ... ],
  "open_count": 41,
  "open_by_severity": { "critical": 2, "high": 9, "medium": 21, "low": 9 }
}
```

### Scoring Configuration
```http
GET /risk-register/config
PUT /risk-register/config        # org admin only
Authorization: Bearer {token}
Content-Type: application/json

{
  "category_taxonomy": ["bias", "drift", "security", "regulatory", "model_risk"],
  "severity_thresholds": { "critical": 20, "high": 12, "medium": 6 }
}
```

Also accepts `likelihood_scale` and `impact_scale`. `GET` includes `is_default`.

---

## Governance Review Boards

Multi-party approval workflows with an immutable decision archive. The full endpoint list and
the governance guarantees are documented in [Governance Review
Boards](/user-guide/governance/).

```http
POST   /governance/review-boards
GET    /governance/review-boards
GET|PUT|DELETE /governance/review-boards/{board_id}
POST   /governance/review-boards/{board_id}/members
POST   /governance/review-boards/requests
POST   /governance/review-boards/requests/{request_id}/submit
POST   /governance/review-boards/requests/{request_id}/decisions
GET    /governance/review-boards/requests/{request_id}/status
POST   /governance/review-boards/requests/{request_id}/finalize
GET    /governance/review-boards/my-reviews
POST   /governance/review-boards/archive/search
POST   /governance/review-boards/archive/export
```

---

## Alerts

!!! warning "Not available yet"
    The alerts REST API is not registered on the backend. Requests to `/api/v1/alerts` and
    `/api/v1/alerts/rules` return `404`.

    The SDK exposes `client.alerts.create()` / `client.alerts.list()` and
    `ModelMonitor.create_alert_rule()` / `get_active_alerts()`, but these call the endpoints
    above and will fail until they ship. Manage alert rules in the dashboard under
    **Observability → Alerts** for now.

    Drift and bias thresholds *are* configurable through the API — see [Drift
    Detection](#drift-detection) and [Bias & Fairness](#bias--fairness). A `high` or
    `critical` result there auto-drafts a [risk register](#risk-register) entry.

---

## Exports & Reports

Report generation lives under `/export/*`. There is no `/reports` endpoint — it returns `404`.

See [Audit & Explanation Reports](/user-guide/reports/) for the dashboard walkthrough, report
categories, and what a report contains — this section is the endpoint reference.

**Formats:** `pdf`, `csv`, `excel`, `json`, `html`, `markdown`.

**Report categories:** `model_performance`, `drift_analysis`, `bias_audit`, `compliance`,
`explainability`, `llm_monitoring`, `risk_register`, `trust_score`, `custom`.

### Generate an Export
```http
POST /export/exports
Authorization: Bearer {token}
Content-Type: application/json

{
  "template_id": "template_uuid",
  "format": "pdf",
  "model_ids": ["model_uuid"],
  "date_from": "2026-07-01",
  "date_to": "2026-07-31"
}
```

Queue several at once with `POST /export/exports/bulk`.

### Check Export Status
```http
GET /export/exports/{export_id}
Authorization: Bearer {token}
```

`status` moves through `pending` → `in_progress` → `completed`, or `failed`. Also
`cancelled` if you cancel it.

### List and Download
```http
GET /export/exports
GET /export/exports/{export_id}/download
Authorization: Bearer {token}
```

### Templates and Configurations
```http
GET|POST       /export/templates
GET|PUT|DELETE /export/templates/{template_id}
GET|POST       /export/configs
GET|PUT|DELETE /export/configs/{config_id}
```

Templates define report contents; configurations define how a report is produced and
delivered. Delivery methods: `download`, `email`, `webhook`, `s3`, `sftp`, `api`.

### Scheduled Reports
```http
GET|POST       /export/scheduled-reports
GET|PUT|DELETE /export/scheduled-reports/{report_id}
POST           /export/scheduled-reports/{report_id}/run
```

```json
{
  "name": "Weekly Performance Summary",
  "template_id": "template_uuid",
  "format": "pdf",
  "cron_expression": "0 9 * * 1",
  "recipients": ["team@company.com"]
}
```

### Third-Party Integrations
```http
GET|POST       /export/integrations
GET|PUT|DELETE /export/integrations/{integration_id}
```

---

## Users & RBAC

### List Users
```http
GET /users?role=admin&status=active
Authorization: Bearer {token}
```

### Create User
```http
POST /users
Authorization: Bearer {token}
Content-Type: application/json

{
  "username": "newuser@example.com",
  "email": "newuser@example.com",
  "password": "secure_password",
  "full_name": "New User",
  "role": "viewer"
}
```

### Update User Role
```http
PUT /users/{user_id}/role
Authorization: Bearer {token}
Content-Type: application/json

{
  "role": "admin"
}
```

### List Roles
```http
GET /roles
Authorization: Bearer {token}
```

### Create Role
```http
POST /roles
Authorization: Bearer {token}
Content-Type: application/json

{
  "name": "data_scientist",
  "description": "Data scientist with model access",
  "permissions": [
    "models:read",
    "models:create",
    "predictions:read",
    "explanations:read"
  ]
}
```

---

## Dashboard

### Get Dashboard Stats
```http
GET /dashboard/stats?time_range=7d
Authorization: Bearer {token}
```

**Response (200 OK):**
```json
{
  "models": {
    "total": 45,
    "active": 38,
    "inactive": 7
  },
  "predictions": {
    "total": 1542000,
    "today": 45000,
    "avg_per_day": 220285
  },
  "alerts": {
    "active": 3,
    "acknowledged": 12,
    "resolved": 45
  },
  "drift": {
    "models_with_drift": 5,
    "high_severity": 2
  }
}
```

### WebSocket Connection
```javascript
// Connect to real-time dashboard updates
const ws = new WebSocket('wss://api.whiteboxxai.com/dashboard/ws?token={token}');

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log('Update:', data);
};
```

---

## Error Handling

### Error Response Format
All errors follow a consistent format:

```json
{
  "error": {
    "code": "MODEL_NOT_FOUND",
    "message": "Model with ID 'model_uuid' not found",
    "details": {
      "model_id": "model_uuid"
    },
    "timestamp": "2025-12-05T15:30:00Z",
    "request_id": "req_xyz"
  }
}
```

### HTTP Status Codes

| Status Code | Meaning |
|-------------|---------|
| 200 | OK - Request successful |
| 201 | Created - Resource created successfully |
| 202 | Accepted - Request accepted for processing |
| 204 | No Content - Successful deletion |
| 400 | Bad Request - Invalid request data |
| 401 | Unauthorized - Missing or invalid token |
| 403 | Forbidden - Insufficient permissions |
| 404 | Not Found - Resource not found |
| 409 | Conflict - Resource already exists |
| 422 | Unprocessable Entity - Validation error |
| 429 | Too Many Requests - Rate limit exceeded |
| 500 | Internal Server Error - Server error |
| 503 | Service Unavailable - Service temporarily unavailable |

### Common Error Codes

| Code | Description |
|------|-------------|
| `AUTHENTICATION_FAILED` | Invalid credentials |
| `TOKEN_EXPIRED` | JWT token has expired |
| `INSUFFICIENT_PERMISSIONS` | User lacks required permissions |
| `MODEL_NOT_FOUND` | Model does not exist |
| `VALIDATION_ERROR` | Request data validation failed |
| `RATE_LIMIT_EXCEEDED` | Too many requests |
| `RESOURCE_CONFLICT` | Resource already exists |
| `PROCESSING_ERROR` | Error processing request |

---

## Rate Limits

- **Standard:** 1000 requests per hour
- **Prediction Logging:** 10,000 requests per hour
- **Batch Operations:** 100 requests per hour
- **Report Generation:** 10 requests per hour

Rate limit headers:
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 950
X-RateLimit-Reset: 1638720000
```

---

## Pagination

List endpoints support pagination:

**Request:**
```http
GET /models?page=2&page_size=20
```

**Response:**
```json
{
  "total": 145,
  "page": 2,
  "page_size": 20,
  "total_pages": 8,
  "has_next": true,
  "has_prev": true,
  "next_page": 3,
  "prev_page": 1,
  "models": [...]
}
```

---

## Versioning

The API uses URL versioning. Current version: **v1**

Future versions will be available at:
- `/api/v2/...`
- `/api/v3/...`

---

## SDK Integration

For easier integration, use the Python SDK:

```python
from whiteboxxai import WhiteBoxXAI

client = WhiteBoxXAI(api_key="your_api_key")

# Register model
model = client.models.register(
    name="my_model",
    model_type="classification"
)

# Log prediction
client.predictions.log(
    model_id=model.id,
    input_data={"feature": 1.5},
    output_data={"prediction": 0}
)
```

See [SDK Documentation](/sdk/) for details.

---

## Support

- **Documentation:** https://docs.whiteboxxai.com
- **API Status:** https://status.whiteboxxai.com
- **Support:** support@whiteboxxai.com

---

*Last Updated: December 5, 2025*
