---
description: WhiteBoxXAI REST API reference — base URL, authentication with API keys, the error envelope, rate limits, Governance Event metering and the X-WhiteBox-Run-Id header, pagination, and every customer-facing endpoint grouped by area.
---

# REST API Reference

Everything in the dashboard and the [Python SDK](index.md) is built on this REST API. Use it from any
language. In Python, the SDK is usually simpler: it handles retries, offline queuing and run grouping
for you.

!!! note "Machine-readable contract"
    This page lists every customer-facing endpoint. A published, machine-readable OpenAPI contract
    with full request and response schemas is planned. Until then, the SDK's method signatures and
    the examples on each guide page are the reference for payload fields.

## Base URL and versioning

```text
https://api.whiteboxxai.com/api/v1
```

Business Cloud tenants use the API address they were given when the tenant was set up. All endpoints
are under `/api/v1`. Additive changes, such as new fields and new endpoints, happen within `v1`; clients
should ignore response fields they don't recognise.

## Authentication

Send an [API key](../account/api-keys.md) as a bearer token on every request:

```bash
curl https://api.whiteboxxai.com/api/v1/models \
  -H "Authorization: Bearer $WHITEBOX_API_KEY"
```

- Keys look like `wbx_live_...`. They're created by administrators in **Settings → API keys**, and
  act with the permissions of the administrator who created them.
- A signed-in session token (about 30 minutes) is also accepted, and is required for actions that
  must be made by a person, such as [voting on a review request](../user-guide/governance.md#vote-with-an-authenticator-code).
- A missing, wrong, revoked or expired credential returns `401`.

Every response carries an `X-Request-ID` header. Include it when you contact support.

## Errors

Errors share one envelope:

```json
{
  "error": "HTTPException",
  "detail": "Model not found",
  "status_code": 404
}
```

| Status | `error` | Meaning |
| --- | --- | --- |
| `400` | `HTTPException` | The request was understood but can't be carried out as sent |
| `401` | `HTTPException` | Missing or invalid credential |
| `403` | `HTTPException` | Not allowed: wrong role, an action that needs a person's own session, or your plan's [Governed AI Systems](../get-started/governance-events.md#what-is-one-governed-ai-system) are all in use |
| `404` | `HTTPException` | Doesn't exist **in your organization**. Records in other organizations are indistinguishable from records that don't exist. |
| `409` | `HTTPException` | Conflict, such as a duplicate name, or a review request that changed after you opened it |
| `422` | `ValidationError` | The body or parameters didn't match the schema; `detail` lists each problem |
| `429` | `QuotaExceeded` | The Free plan's monthly [Governance Events](../get-started/governance-events.md#when-you-reach-the-limit) are used up, or a batch is larger than what remains |
| `429` | `RateLimitExceeded` | Too many requests in the current window; wait for `Retry-After` |
| `5xx` | | A server error; safe to retry with back-off |

## Rate limits

The default rate limit is **100 requests per 60 seconds** per client. Responses include
`X-RateLimit-Limit`, `X-RateLimit-Remaining` and `X-RateLimit-Reset`. A `429 RateLimitExceeded`
response also includes `Retry-After`, in seconds. Prefer batch endpoints for volume.

## Metering and run grouping

Requests authenticated with an API key that record AI activity (logging predictions, LLM calls, RAG
retrievals and safety scans, generating explanations, running evaluations, starting agent workflows)
count as [Governance Events](../get-started/governance-events.md). Reads, configuration, reports and
dashboard use don't. Events are counted only for successful responses.

Group the records of one AI run with the optional header:

```text
X-WhiteBox-Run-Id: ticket-4821
```

Up to 128 letters, digits, `.`, `_`, `:` or `-`. Predictions, LLM calls, RAG retrievals and safety scans
sharing a run id cost one event per 100 records. A value in any other format is ignored.

## Pagination

List endpoints take `skip` (offset, default `0`) and `limit` (page size). The maximum `limit` varies by
endpoint and is usually 100 or 1,000. A few newer endpoints use `limit` and `offset`. Time-range filters
take ISO 8601 timestamps, for example `start_time=2026-10-01T00:00:00Z`.

## Common requests

=== "Register a model"

    ```bash
    curl -X POST https://api.whiteboxxai.com/api/v1/models \
      -H "Authorization: Bearer $WHITEBOX_API_KEY" -H "Content-Type: application/json" \
      -d '{
        "name": "credit_scoring",
        "version": "1.0.0",
        "model_type": "xgboost",
        "model_framework": "xgboost",
        "features": ["income", "utilisation", "tenure_months"],
        "baseline_metrics": {"accuracy": 0.94}
      }'
    ```

    `model_type` is one of `xgboost`, `random_forest`, `neural_network`, `linear_model`, `llm`, `custom`.

=== "Log a prediction"

    ```bash
    curl -X POST https://api.whiteboxxai.com/api/v1/predictions/log \
      -H "Authorization: Bearer $WHITEBOX_API_KEY" -H "Content-Type: application/json" \
      -d '{
        "model_id": "<model-id>",
        "input_data": {"income": 52000, "utilisation": 0.31, "tenure_months": 18},
        "output_data": {"prediction": 0, "probability": 0.12},
        "latency_ms": 4.2
      }'
    ```

=== "Log an LLM call"

    ```bash
    curl -X POST https://api.whiteboxxai.com/api/v1/llm/logs \
      -H "Authorization: Bearer $WHITEBOX_API_KEY" -H "Content-Type: application/json" \
      -H "X-WhiteBox-Run-Id: ticket-4821" \
      -d '{
        "provider": "xai",
        "model_name": "grok-4",
        "prompt": "Summarise this complaint...",
        "completion": "The customer reports...",
        "latency_ms": 712,
        "prompt_tokens": 210,
        "completion_tokens": 64
      }'
    ```

=== "Check drift"

    ```bash
    # Ad-hoc check, not stored
    curl -X POST "https://api.whiteboxxai.com/api/v1/drift/detect/<model-id>?window_size=1000" \
      -H "Authorization: Bearer $WHITEBOX_API_KEY"

    # Stored drift reports for a model
    curl https://api.whiteboxxai.com/api/v1/drift/reports/<model-id> \
      -H "Authorization: Bearer $WHITEBOX_API_KEY"
    ```

=== "Generate a report"

    ```bash
    curl -X POST "https://api.whiteboxxai.com/api/v1/report-types/model_card/generate?format=pdf&model_id=<model-id>" \
      -H "Authorization: Bearer $WHITEBOX_API_KEY"
    ```

    See [Report types reference](../user-guide/report-engine.md).

## Endpoints

Grouped by area. Paths are relative to `https://api.whiteboxxai.com`. Guides for each area are linked
from the [Feature reference](../user-guide/features.md#where-each-feature-lives).

### Models

| Method | Path | Summary |
| --- | --- | --- |
| `GET` | `/api/v1/models` | List Models |
| `POST` | `/api/v1/models` | Register Model |
| `GET` | `/api/v1/models/{model_id}` | Get Model |
| `PATCH` | `/api/v1/models/{model_id}` | Update Model |
| `DELETE` | `/api/v1/models/{model_id}` | Delete Model |
| `POST` | `/api/v1/models/{model_id}/archive` | Archive Model |
| `PATCH` | `/api/v1/models/{model_id}/baseline` | Update Baseline |
| `GET` | `/api/v1/models/{model_id}/dossier` | Get Model Dossier |
| `POST` | `/api/v1/models/{model_id}/restore` | Restore Model |
| `PATCH` | `/api/v1/models/{model_id}/status` | Update Model Status |
| `GET` | `/api/v1/models/{model_name}/latest` | Get Latest Version |
| `GET` | `/api/v1/models/{model_name}/versions` | Get Model Versions |

### Predictions

| Method | Path | Summary |
| --- | --- | --- |
| `POST` | `/api/v1/predictions/log` | Log a single prediction |
| `POST` | `/api/v1/predictions/log/batch` | Log multiple predictions in batch |
| `GET` | `/api/v1/predictions/models/{model_id}/recent` | Get recent predictions for a model |
| `GET` | `/api/v1/predictions/models/{model_id}/stats` | Get prediction statistics for a model |
| `POST` | `/api/v1/predictions/query` | Query predictions with filters |
| `GET` | `/api/v1/predictions/{prediction_id}` | Get prediction by ID |

### Metrics

| Method | Path | Summary |
| --- | --- | --- |
| `POST` | `/api/v1/metrics` | Create Metric |
| `POST` | `/api/v1/metrics/batch` | Create Metrics Batch |
| `POST` | `/api/v1/metrics/calculate/classification` | Calculate Classification Metrics |
| `POST` | `/api/v1/metrics/calculate/regression` | Calculate Regression Metrics |
| `GET` | `/api/v1/metrics/{model_id}` | Get Model Metrics |
| `DELETE` | `/api/v1/metrics/{model_id}` | Delete Model Metrics |
| `GET` | `/api/v1/metrics/{model_id}/aggregate` | Get Aggregated Metrics |
| `GET` | `/api/v1/metrics/{model_id}/latest/{metric_type}` | Get Latest Metric |
| `GET` | `/api/v1/metrics/{model_id}/rolling/{metric_type}` | Get Rolling Statistics |
| `GET` | `/api/v1/metrics/{model_id}/summary` | Get Model Metrics Summary |
| `GET` | `/api/v1/metrics/{model_id}/timeseries/{metric_type}` | Get Metric Timeseries |
| `GET` | `/api/v1/metrics/{model_id}/trend/{metric_type}` | Get Metric Trend |

### Explanations

| Method | Path | Summary |
| --- | --- | --- |
| `POST` | `/api/v1/explanations/compare` | Compare Explanations |
| `POST` | `/api/v1/explanations/config` | Create Or Update Config |
| `GET` | `/api/v1/explanations/config/{model_id}` | Get Config |
| `DELETE` | `/api/v1/explanations/config/{model_id}` | Delete Config |
| `POST` | `/api/v1/explanations/generate` | Generate Explanation |
| `POST` | `/api/v1/explanations/generate/async` | Generate Explanation Async |
| `POST` | `/api/v1/explanations/generate/bulk` | Generate Bulk Explanations |
| `GET` | `/api/v1/explanations/model/{model_id}` | Get Model Explanations |
| `GET` | `/api/v1/explanations/model/{model_id}/stats` | Get Explanation Stats |
| `GET` | `/api/v1/explanations/prediction/{prediction_id}` | Get Prediction Explanation |
| `POST` | `/api/v1/explanations/visualizations/multi` | Generate Multi Visualization |
| `POST` | `/api/v1/explanations/visualizations/single` | Generate Single Visualization |
| `GET` | `/api/v1/explanations/{explanation_id}` | Get Explanation |

### Drift

| Method | Path | Summary |
| --- | --- | --- |
| `POST` | `/api/v1/concept-drift/detect/{model_id}` | Detect Concept Drift |
| `POST` | `/api/v1/concept-drift/reports` | Create Concept Drift Report |
| `GET` | `/api/v1/concept-drift/reports/{model_id}` | Get Concept Drift Reports |
| `GET` | `/api/v1/concept-drift/reports/{model_id}/{report_id}` | Get Concept Drift Report |
| `GET` | `/api/v1/concept-drift/trend/{model_id}` | Get Concept Drift Trend |
| `GET` | `/api/v1/drift-config/presets` | Get Sensitivity Presets |
| `GET` | `/api/v1/drift-config/presets/{preset_name}` | Get Sensitivity Preset |
| `GET` | `/api/v1/drift-config/{model_id}` | Get Drift Config |
| `POST` | `/api/v1/drift-config/{model_id}` | Create Drift Config |
| `PUT` | `/api/v1/drift-config/{model_id}` | Update Drift Config |
| `DELETE` | `/api/v1/drift-config/{model_id}` | Delete Drift Config |
| `POST` | `/api/v1/drift-config/{model_id}/default` | Create Default Config |
| `POST` | `/api/v1/drift-config/{model_id}/sensitivity/{sensitivity}` | Apply Sensitivity Preset |
| `POST` | `/api/v1/drift/detect/{model_id}` | Detect Drift |
| `POST` | `/api/v1/drift/reports` | Create Drift Report |
| `GET` | `/api/v1/drift/reports/{model_id}` | Get Drift Reports |
| `GET` | `/api/v1/drift/reports/{model_id}/{report_id}` | Get Drift Report |
| `GET` | `/api/v1/drift/trend/{model_id}` | Get Drift Trend |

### Fairness

| Method | Path | Summary |
| --- | --- | --- |
| `POST` | `/api/v1/fairness/audit` | Run Bias Audit |
| `GET` | `/api/v1/fairness/audits` | List Audits |
| `GET` | `/api/v1/fairness/audits/{audit_id}` | Get Audit Results |
| `DELETE` | `/api/v1/fairness/models/{model_id}/audits/cleanup` | Cleanup Old Audits |
| `GET` | `/api/v1/fairness/models/{model_id}/bias-history` | Get Bias History |
| `GET` | `/api/v1/fairness/models/{model_id}/latest-audit` | Get Latest Audit |
| `GET` | `/api/v1/fairness/models/{model_id}/metrics/{metric_type}/history` | Get Metric History |

### LLM monitoring

| Method | Path | Summary |
| --- | --- | --- |
| `GET` | `/api/v1/llm/alerts/cost-threshold` | Check Cost Threshold |
| `GET` | `/api/v1/llm/alerts/error-rate` | Check Error Rate |
| `GET` | `/api/v1/llm/alerts/latency-threshold` | Check Latency Threshold |
| `GET` | `/api/v1/llm/costs/breakdown` | Get Cost Breakdown |
| `POST` | `/api/v1/llm/logs` | Create Llm Log |
| `POST` | `/api/v1/llm/logs/batch` | Create Llm Logs Batch |
| `DELETE` | `/api/v1/llm/logs/cleanup` | Cleanup Old Logs |
| `POST` | `/api/v1/llm/logs/query` | Query Llm Logs |
| `GET` | `/api/v1/llm/logs/recent` | Get Recent Logs |
| `GET` | `/api/v1/llm/logs/session/{session_id}` | Get Logs By Session |
| `GET` | `/api/v1/llm/logs/{log_id}` | Get Llm Log |
| `GET` | `/api/v1/llm/performance` | Get Performance Metrics |
| `GET` | `/api/v1/llm/stats` | Get Usage Stats |
| `GET` | `/api/v1/llm/trends/costs` | Get Cost Trend |
| `GET` | `/api/v1/llm/trends/tokens` | Get Token Usage Trend |
| `POST` | `/api/v1/llm/usage-stats` | Get Usage Stats Detailed |

### LLM explainability

| Method | Path | Summary |
| --- | --- | --- |
| `POST` | `/api/v1/llm-xai/attention` | Analyze Attention |
| `GET` | `/api/v1/llm-xai/attention/visualize/{explanation_id}` | Visualize Attention |
| `POST` | `/api/v1/llm-xai/batch-analyze` | Batch Analyze |
| `POST` | `/api/v1/llm-xai/counterfactuals` | Generate Counterfactuals |
| `POST` | `/api/v1/llm-xai/debug-prompt` | Debug Prompt |
| `GET` | `/api/v1/llm-xai/explanations` | Get Explanations |
| `GET` | `/api/v1/llm-xai/explanations/{explanation_id}` | Get Explanation |
| `GET` | `/api/v1/llm-xai/prompt-analyses` | Get Prompt Analyses |
| `GET` | `/api/v1/llm-xai/prompt-analyses/{analysis_id}` | Get Prompt Analysis |
| `POST` | `/api/v1/llm-xai/sensitivity` | Analyze Sensitivity |
| `GET` | `/api/v1/llm-xai/stats` | Get Xai Stats |
| `POST` | `/api/v1/llm-xai/token-importance` | Analyze Token Importance |
| `GET` | `/api/v1/llm-xai/token-importance/visualize/{explanation_id}` | Visualize Token Importance |

### RAG

| Method | Path | Summary |
| --- | --- | --- |
| `GET` | `/api/v1/rag/evaluations` | List Evaluations |
| `POST` | `/api/v1/rag/evaluations` | Create Evaluation |
| `GET` | `/api/v1/rag/evaluations/{evaluation_id}` | Get Evaluation |
| `GET` | `/api/v1/rag/metrics/precision` | Get Precision Metrics |
| `GET` | `/api/v1/rag/metrics/relevance` | Get Relevance Metrics |
| `GET` | `/api/v1/rag/retrievals` | Query Retrieval Logs |
| `POST` | `/api/v1/rag/retrievals` | Log Retrieval |
| `GET` | `/api/v1/rag/retrievals/{log_id}` | Get Retrieval Log |
| `GET` | `/api/v1/rag/stats` | Get Rag Statistics |
| `GET` | `/api/v1/rag/trends` | Get Rag Trend |

### Safety

| Method | Path | Summary |
| --- | --- | --- |
| `POST` | `/api/v1/safety/analyze` | Analyze Content |
| `POST` | `/api/v1/safety/analyze/batch` | Analyze Batch |
| `GET` | `/api/v1/safety/scores` | Get Safety Scores |
| `GET` | `/api/v1/safety/scores/{score_id}` | Get Safety Score |
| `GET` | `/api/v1/safety/stats` | Get Safety Statistics |
| `GET` | `/api/v1/safety/thresholds` | List Thresholds |
| `POST` | `/api/v1/safety/thresholds` | Create Threshold |
| `GET` | `/api/v1/safety/thresholds/{threshold_id}` | Get Threshold |
| `PATCH` | `/api/v1/safety/thresholds/{threshold_id}` | Update Threshold |
| `DELETE` | `/api/v1/safety/thresholds/{threshold_id}` | Delete Threshold |
| `GET` | `/api/v1/safety/trends` | Get Safety Trend |

### Prompts

| Method | Path | Summary |
| --- | --- | --- |
| `GET` | `/api/v1/prompts` | List governed prompt templates |
| `POST` | `/api/v1/prompts` | Create a governed prompt template |
| `GET` | `/api/v1/prompts/{template_id}` | Get prompt template details |
| `POST` | `/api/v1/prompts/{template_id}/rollback` | Execute an emergency rollback on a deployed prompt version |
| `GET` | `/api/v1/prompts/{template_id}/versions` | List all versions of a prompt template |
| `POST` | `/api/v1/prompts/{template_id}/versions` | Create a new version for a prompt template |
| `GET` | `/api/v1/prompts/{template_id}/versions/{version_id}` | Get prompt version details |
| `POST` | `/api/v1/prompts/{template_id}/versions/{version_id}/advance-stage` | Advance prompt version lifecycle stage |
| `POST` | `/api/v1/prompts/{template_id}/versions/{version_id}/deploy` | Promote prompt version into production deployment |
| `GET` | `/api/v1/prompts/{template_id}/versions/{version_id}/drift-reports` | List historical drift reports |
| `POST` | `/api/v1/prompts/{template_id}/versions/{version_id}/drift-reports` | Compute prompt drift report |
| `GET` | `/api/v1/prompts/{template_id}/versions/{version_id}/drift-reports/{report_id}` | Get prompt drift report details |
| `GET` | `/api/v1/prompts/{template_id}/versions/{version_id}/incidents` | List incidents and alerts tied to a prompt version |

### Evaluations

| Method | Path | Summary |
| --- | --- | --- |
| `GET` | `/api/v1/evaluations/golden-datasets` | List golden benchmark datasets |
| `POST` | `/api/v1/evaluations/golden-datasets` | Create a golden benchmark dataset |
| `GET` | `/api/v1/evaluations/golden-datasets/{dataset_id}` | Get a golden dataset with its benchmark rows |
| `PUT` | `/api/v1/evaluations/golden-datasets/{dataset_id}` | Update a golden dataset |
| `DELETE` | `/api/v1/evaluations/golden-datasets/{dataset_id}` | Delete a golden dataset |
| `POST` | `/api/v1/evaluations/golden-datasets/{dataset_id}/rows` | Add benchmark rows to a golden dataset |
| `DELETE` | `/api/v1/evaluations/golden-datasets/{dataset_id}/rows/{row_id}` | Delete a row from a golden dataset |
| `GET` | `/api/v1/evaluations/prompt` | List prompt evaluations |
| `POST` | `/api/v1/evaluations/prompt` | Run or record a prompt evaluation harness against a golden dataset |
| `GET` | `/api/v1/evaluations/prompt-templates` | List prompt templates |
| `POST` | `/api/v1/evaluations/prompt-templates` | Create a prompt template entity |
| `GET` | `/api/v1/evaluations/prompt-templates/{template_id}` | Get prompt template by ID |
| `GET` | `/api/v1/evaluations/prompt-templates/{template_id}/versions` | List all versions of a prompt template |
| `POST` | `/api/v1/evaluations/prompt-templates/{template_id}/versions` | Create a new version for a prompt template |
| `GET` | `/api/v1/evaluations/prompt-versions/{version_id}` | Get prompt version by ID |
| `POST` | `/api/v1/evaluations/prompt/compare` | Compare two prompt evaluations to detect regressions across versions |
| `GET` | `/api/v1/evaluations/prompt/{evaluation_id}` | Get prompt evaluation details and row results |

### Agent workflows

| Method | Path | Summary |
| --- | --- | --- |
| `GET` | `/api/v1/workflows/multi-agent` | List Workflows |
| `POST` | `/api/v1/workflows/multi-agent/start` | Create And Start Workflow |
| `PATCH` | `/api/v1/workflows/multi-agent/tasks/{task_id}` | Update Task Status |
| `GET` | `/api/v1/workflows/multi-agent/{workflow_id}` | Get Workflow |
| `DELETE` | `/api/v1/workflows/multi-agent/{workflow_id}` | Delete Workflow |
| `GET` | `/api/v1/workflows/multi-agent/{workflow_id}/agents` | List Workflow Agents |
| `POST` | `/api/v1/workflows/multi-agent/{workflow_id}/agents` | Register Agent |
| `GET` | `/api/v1/workflows/multi-agent/{workflow_id}/analytics` | Get Workflow Analytics |
| `GET` | `/api/v1/workflows/multi-agent/{workflow_id}/bottlenecks` | Get Bottlenecks |
| `POST` | `/api/v1/workflows/multi-agent/{workflow_id}/complete` | Complete Workflow |
| `GET` | `/api/v1/workflows/multi-agent/{workflow_id}/cost-breakdown` | Get Cost Breakdown |
| `POST` | `/api/v1/workflows/multi-agent/{workflow_id}/executions` | Create Agent Execution |
| `GET` | `/api/v1/workflows/multi-agent/{workflow_id}/interactions` | List Workflow Interactions |
| `POST` | `/api/v1/workflows/multi-agent/{workflow_id}/interactions` | Log Agent Interaction |
| `POST` | `/api/v1/workflows/multi-agent/{workflow_id}/start` | Start Workflow |
| `GET` | `/api/v1/workflows/multi-agent/{workflow_id}/tasks` | List Workflow Tasks |
| `POST` | `/api/v1/workflows/multi-agent/{workflow_id}/tasks` | Create Task |
| `GET` | `/api/v1/workflows/multi-agent/{workflow_id}/timeline` | Get Workflow Timeline |

### Alerts

| Method | Path | Summary |
| --- | --- | --- |
| `GET` | `/api/v1/alerts/instances` | Get Alerts |
| `GET` | `/api/v1/alerts/instances/{alert_id}` | Get Alert |
| `POST` | `/api/v1/alerts/instances/{alert_id}/acknowledge` | Acknowledge Alert |
| `POST` | `/api/v1/alerts/instances/{alert_id}/resolve` | Resolve Alert |
| `POST` | `/api/v1/alerts/instances/{alert_id}/snooze` | Snooze Alert |
| `GET` | `/api/v1/alerts/rules` | Get Alert Rules |
| `POST` | `/api/v1/alerts/rules` | Create Alert Rule |
| `GET` | `/api/v1/alerts/rules/{rule_id}` | Get Alert Rule |
| `PATCH` | `/api/v1/alerts/rules/{rule_id}` | Update Alert Rule |
| `DELETE` | `/api/v1/alerts/rules/{rule_id}` | Delete Alert Rule |
| `POST` | `/api/v1/alerts/rules/{rule_id}/evaluate` | Evaluate Alert Rule |
| `GET` | `/api/v1/alerts/statistics` | Get Alert Statistics |

### Alert intelligence

| Method | Path | Summary |
| --- | --- | --- |
| `POST` | `/api/v1/alert-intelligence/alerts/{alert_id}/correlations` | Detect Alert Correlations |
| `POST` | `/api/v1/alert-intelligence/alerts/{alert_id}/priority` | Calculate Alert Priority |
| `PATCH` | `/api/v1/alert-intelligence/alerts/{alert_id}/priority` | Adjust Alert Priority |
| `POST` | `/api/v1/alert-intelligence/alerts/{alert_id}/root-cause` | Analyze Alert Root Cause |
| `GET` | `/api/v1/alert-intelligence/correlations/clusters` | Get Correlation Clusters |
| `POST` | `/api/v1/alert-intelligence/fatigue-reduction/apply` | Apply Fatigue Reduction |
| `GET` | `/api/v1/alert-intelligence/fatigue-reduction/recommendations` | Get Throttle Recommendations |
| `POST` | `/api/v1/alert-intelligence/grouping/ml` | Ml Based Grouping |
| `POST` | `/api/v1/alert-intelligence/grouping/temporal` | Temporal Grouping |
| `GET` | `/api/v1/alert-intelligence/patterns` | Detect Alert Patterns |
| `GET` | `/api/v1/alert-intelligence/patterns/{pattern_id}` | Get Pattern |
| `PATCH` | `/api/v1/alert-intelligence/patterns/{pattern_id}/deactivate` | Deactivate Pattern |
| `GET` | `/api/v1/alert-intelligence/priority/distribution` | Get Priority Distribution |
| `GET` | `/api/v1/alert-intelligence/priority/queue` | Get Priority Queue |
| `GET` | `/api/v1/alert-intelligence/root-causes/similar` | Get Similar Incidents |
| `POST` | `/api/v1/alert-intelligence/root-causes/{root_cause_id}/verify` | Verify Root Cause |
| `POST` | `/api/v1/alert-intelligence/trends/enhanced` | Enhanced Trend Analysis |
| `POST` | `/api/v1/alert-intelligence/trends/forecast` | Forecast Alert Volume |
| `POST` | `/api/v1/alert-intelligence/trends/predict` | Predict Alert Trends |
| `GET` | `/api/v1/alert-intelligence/trends/seasonality` | Detect Seasonality |

### Alert management

| Method | Path | Summary |
| --- | --- | --- |
| `POST` | `/api/v1/alert-management/alerts/bulk-action` | Bulk Alert Action |
| `POST` | `/api/v1/alert-management/alerts/{alert_id}/acknowledge` | Acknowledge Alert |
| `GET` | `/api/v1/alert-management/alerts/{alert_id}/comments` | Get Alert Comments |
| `POST` | `/api/v1/alert-management/alerts/{alert_id}/comments` | Create Alert Comment |
| `GET` | `/api/v1/alert-management/alerts/{alert_id}/history` | Get Alert History |
| `POST` | `/api/v1/alert-management/alerts/{alert_id}/resolve` | Resolve Alert |
| `POST` | `/api/v1/alert-management/alerts/{alert_id}/snooze` | Snooze Alert |
| `GET` | `/api/v1/alert-management/analytics` | Get Alert Analytics |
| `GET` | `/api/v1/alert-management/analytics/fatigue` | Detect Alert Fatigue |
| `GET` | `/api/v1/alert-management/analytics/grouping` | Get Alert Grouping |
| `POST` | `/api/v1/alert-management/analytics/trend` | Get Alert Trend |
| `GET` | `/api/v1/alert-management/analytics/velocity` | Get Alert Velocity |
| `GET` | `/api/v1/alert-management/escalation-policies` | List Escalation Policies |
| `POST` | `/api/v1/alert-management/escalation-policies` | Create Escalation Policy |
| `GET` | `/api/v1/alert-management/escalation-policies/{policy_id}` | Get Escalation Policy |
| `PATCH` | `/api/v1/alert-management/escalation-policies/{policy_id}` | Update Escalation Policy |
| `DELETE` | `/api/v1/alert-management/escalation-policies/{policy_id}` | Delete Escalation Policy |
| `POST` | `/api/v1/alert-management/maintenance/process-escalations` | Process Pending Escalations |
| `POST` | `/api/v1/alert-management/maintenance/unsnooze-alerts` | Unsnooze Expired Alerts |

### Notifications

| Method | Path | Summary |
| --- | --- | --- |
| `GET` | `/api/v1/notifications/channels` | List Notification Channels |
| `POST` | `/api/v1/notifications/channels` | Create Notification Channel |
| `GET` | `/api/v1/notifications/channels/{channel_id}` | Get Notification Channel |
| `PATCH` | `/api/v1/notifications/channels/{channel_id}` | Update Notification Channel |
| `DELETE` | `/api/v1/notifications/channels/{channel_id}` | Delete Notification Channel |
| `GET` | `/api/v1/notifications/channels/{channel_id}/statistics` | Get Channel Statistics |
| `POST` | `/api/v1/notifications/channels/{channel_id}/test` | Test Notification Channel |
| `GET` | `/api/v1/notifications/deliveries` | List Notification Deliveries |
| `GET` | `/api/v1/notifications/deliveries/{delivery_id}` | Get Notification Delivery |
| `GET` | `/api/v1/notifications/preferences` | Get Notification Preferences |
| `POST` | `/api/v1/notifications/preferences` | Create Notification Preferences |
| `PATCH` | `/api/v1/notifications/preferences` | Update Notification Preferences |
| `DELETE` | `/api/v1/notifications/preferences` | Delete Notification Preferences |
| `POST` | `/api/v1/notifications/retry-failed` | Retry Failed Deliveries |

### Trust score

| Method | Path | Summary |
| --- | --- | --- |
| `GET` | `/api/v1/trust-score/config` | Get Trust Score Config |
| `PUT` | `/api/v1/trust-score/config` | Update Trust Score Config |
| `GET` | `/api/v1/trust-score/portfolio` | Get Trust Score Portfolio |
| `GET` | `/api/v1/trust-score/{model_id}` | Get Trust Score Detail |
| `GET` | `/api/v1/trust-score/{model_id}/history` | Get Trust Score History |

### Risk register

| Method | Path | Summary |
| --- | --- | --- |
| `GET` | `/api/v1/risk-register` | List Risk Entries |
| `POST` | `/api/v1/risk-register` | Create Risk Entry |
| `GET` | `/api/v1/risk-register/config` | Get Scoring Config |
| `PUT` | `/api/v1/risk-register/config` | Update Scoring Config |
| `GET` | `/api/v1/risk-register/portfolio` | Get Risk Portfolio |
| `GET` | `/api/v1/risk-register/{entry_id}` | Get Risk Entry |
| `PATCH` | `/api/v1/risk-register/{entry_id}` | Update Risk Entry |
| `GET` | `/api/v1/risk-register/{entry_id}/history` | Get Risk Entry History |

### Governance and review boards

| Method | Path | Summary |
| --- | --- | --- |
| `GET` | `/api/v1/governance/accountability/approvals/pending` | Get Pending Approvals |
| `POST` | `/api/v1/governance/accountability/approvals/request` | Request Approval |
| `POST` | `/api/v1/governance/accountability/approvals/{workflow_id}/deny` | Deny Approval |
| `POST` | `/api/v1/governance/accountability/approvals/{workflow_id}/grant` | Grant Approval |
| `POST` | `/api/v1/governance/accountability/overrides` | Override Decision |
| `GET` | `/api/v1/governance/accountability/responsibility-matrix` | Get Responsibility Matrix |
| `POST` | `/api/v1/governance/accountability/responsibility-matrix` | Create Responsibility Matrix |
| `GET` | `/api/v1/governance/accountability/workflows` | Get Workflows |
| `POST` | `/api/v1/governance/accountability/workflows` | Create Workflow |
| `POST` | `/api/v1/governance/accountability/workflows/{workflow_id}/escalate` | Escalate Workflow |
| `GET` | `/api/v1/governance/dashboard` | Get Governance Dashboard |
| `POST` | `/api/v1/governance/privacy/anonymize` | Anonymize Data |
| `GET` | `/api/v1/governance/privacy/assessments` | Get Privacy Assessments |
| `POST` | `/api/v1/governance/privacy/assessments` | Create Privacy Assessment |
| `POST` | `/api/v1/governance/privacy/detect-pii` | Detect Pii |
| `POST` | `/api/v1/governance/privacy/impact-assessments` | Conduct Privacy Impact Assessment |
| `POST` | `/api/v1/governance/privacy/minimize-data` | Analyze Data Minimization |
| `GET` | `/api/v1/governance/review-boards` | List Boards |
| `POST` | `/api/v1/governance/review-boards` | Create Board |
| `POST` | `/api/v1/governance/review-boards/ai/generate-template` | Generate Ai Template |
| `POST` | `/api/v1/governance/review-boards/ai/suggest-best-practices` | Suggest Best Practices |
| `POST` | `/api/v1/governance/review-boards/archive/export` | Export Decision Archive |
| `GET` | `/api/v1/governance/review-boards/archive/facets` | Get Decision Facets |
| `GET` | `/api/v1/governance/review-boards/archive/recent` | Get Recent Decisions |
| `POST` | `/api/v1/governance/review-boards/archive/search` | Search Decision Archive |
| `GET` | `/api/v1/governance/review-boards/archive/{decision_id}` | Get Decision Archive |
| `GET` | `/api/v1/governance/review-boards/my-reviews` | Get My Review Tasks |
| `PATCH` | `/api/v1/governance/review-boards/participants/{participant_id}` | Update Raci Participant |
| `DELETE` | `/api/v1/governance/review-boards/participants/{participant_id}` | Remove Raci Participant |
| `GET` | `/api/v1/governance/review-boards/raci-grid` | Get Raci Grid |
| `GET` | `/api/v1/governance/review-boards/requests` | List Review Requests |
| `POST` | `/api/v1/governance/review-boards/requests` | Create Review Request |
| `GET` | `/api/v1/governance/review-boards/requests/{request_id}` | Get Review Request |
| `PUT` | `/api/v1/governance/review-boards/requests/{request_id}` | Update Review Request |
| `POST` | `/api/v1/governance/review-boards/requests/{request_id}/attachments` | Add Attachment |
| `GET` | `/api/v1/governance/review-boards/requests/{request_id}/attachments/{attachment_id}/download` | Download Attachment |
| `GET` | `/api/v1/governance/review-boards/requests/{request_id}/comments` | Get Comments |
| `POST` | `/api/v1/governance/review-boards/requests/{request_id}/comments` | Add Comment |
| `GET` | `/api/v1/governance/review-boards/requests/{request_id}/decisions` | Get Decisions |
| `POST` | `/api/v1/governance/review-boards/requests/{request_id}/decisions` | Submit Decision |
| `POST` | `/api/v1/governance/review-boards/requests/{request_id}/finalize` | Finalize Review |
| `GET` | `/api/v1/governance/review-boards/requests/{request_id}/participants` | Get Raci Participants |
| `POST` | `/api/v1/governance/review-boards/requests/{request_id}/participants` | Assign Raci Participant |
| `GET` | `/api/v1/governance/review-boards/requests/{request_id}/status` | Get Vote Status |
| `POST` | `/api/v1/governance/review-boards/requests/{request_id}/submit` | Submit Review Request |
| `POST` | `/api/v1/governance/review-boards/requests/{request_id}/withdraw` | Withdraw Review Request |
| `GET` | `/api/v1/governance/review-boards/schedules` | List Automated Schedules |
| `POST` | `/api/v1/governance/review-boards/schedules` | Create Automated Schedule |
| `GET` | `/api/v1/governance/review-boards/schedules/{schedule_id}` | Get Automated Schedule |
| `PUT` | `/api/v1/governance/review-boards/schedules/{schedule_id}` | Update Automated Schedule |
| `DELETE` | `/api/v1/governance/review-boards/schedules/{schedule_id}` | Delete Automated Schedule |
| `POST` | `/api/v1/governance/review-boards/schedules/{schedule_id}/execute` | Execute Automated Schedule |
| `GET` | `/api/v1/governance/review-boards/{board_id}` | Get Board |
| `PUT` | `/api/v1/governance/review-boards/{board_id}` | Update Board |
| `DELETE` | `/api/v1/governance/review-boards/{board_id}` | Delete Board |
| `GET` | `/api/v1/governance/review-boards/{board_id}/dashboard` | Get Board Dashboard |
| `GET` | `/api/v1/governance/review-boards/{board_id}/members` | Get Board Members |
| `POST` | `/api/v1/governance/review-boards/{board_id}/members` | Add Board Member |
| `PUT` | `/api/v1/governance/review-boards/{board_id}/members/{member_id}` | Update Board Member |
| `DELETE` | `/api/v1/governance/review-boards/{board_id}/members/{member_id}` | Remove Board Member |
| `POST` | `/api/v1/governance/security/detect-adversarial` | Detect Adversarial Attack |
| `GET` | `/api/v1/governance/security/incidents` | Get Security Incidents |
| `POST` | `/api/v1/governance/security/incidents` | Create Security Incident |
| `PATCH` | `/api/v1/governance/security/incidents/{incident_id}` | Update Security Incident |
| `GET` | `/api/v1/governance/security/robustness-tests` | Get Robustness Tests |
| `POST` | `/api/v1/governance/security/robustness-tests` | Create Robustness Test |
| `GET` | `/api/v1/governance/transparency/logs` | Get Transparency Logs |
| `POST` | `/api/v1/governance/transparency/logs` | Create Transparency Log |
| `GET` | `/api/v1/governance/transparency/models/{model_id}/behavior` | Get Model Behavior Summary |
| `GET` | `/api/v1/governance/transparency/models/{model_id}/explanation-coverage` | Get Explanation Coverage |
| `POST` | `/api/v1/governance/transparency/reports/stakeholder` | Generate Stakeholder Report |

### Policy-as-Code

| Method | Path | Summary |
| --- | --- | --- |
| `GET` | `/api/v1/policy` | Get Policy |
| `POST` | `/api/v1/policy/evaluate/{model_id}` | Evaluate Model Policy |
| `POST` | `/api/v1/policy/sync` | Sync Policy |

### Compliance: frameworks, controls and evidence

| Method | Path | Summary |
| --- | --- | --- |
| `GET` | `/api/v1/compliance/adoption-proposals` | List Adoption Proposals |
| `POST` | `/api/v1/compliance/adoption-proposals/{proposal_id}/confirm` | Confirm Adoption Proposal |
| `POST` | `/api/v1/compliance/adoption-proposals/{proposal_id}/reject` | Reject Adoption Proposal |
| `GET` | `/api/v1/compliance/attestations` | List Attestations |
| `POST` | `/api/v1/compliance/attestations` | Create Attestation |
| `GET` | `/api/v1/compliance/attestations/{attestation_id}` | Get Attestation |
| `PUT` | `/api/v1/compliance/attestations/{attestation_id}` | Update Attestation |
| `POST` | `/api/v1/compliance/attestations/{attestation_id}/controls` | Add Control |
| `PUT` | `/api/v1/compliance/attestations/{attestation_id}/controls/{control_id}` | Update Control |
| `POST` | `/api/v1/compliance/attestations/{attestation_id}/controls/{control_id}/check` | Run Automated Check |
| `POST` | `/api/v1/compliance/attestations/{attestation_id}/scorecard` | Generate Scorecard |
| `GET` | `/api/v1/compliance/audit/events` | List Audit Events |
| `POST` | `/api/v1/compliance/audit/events` | Log Audit Event |
| `GET` | `/api/v1/compliance/audit/events/{event_id}` | Get Audit Event |
| `POST` | `/api/v1/compliance/audit/findings` | Create Audit Finding |
| `GET` | `/api/v1/compliance/audit/findings/{finding_id}` | Get Audit Finding |
| `POST` | `/api/v1/compliance/audit/findings/{finding_id}/resolve` | Resolve Audit Finding |
| `GET` | `/api/v1/compliance/audit/reports` | List Audit Reports |
| `POST` | `/api/v1/compliance/audit/reports` | Generate Audit Report |
| `GET` | `/api/v1/compliance/audit/reports/{report_id}` | Get Audit Report |
| `GET` | `/api/v1/compliance/calendar` | Get Calendar |
| `GET` | `/api/v1/compliance/coverage/summary` | Get Coverage Summary |
| `POST` | `/api/v1/compliance/datasheets` | Create Datasheet |
| `GET` | `/api/v1/compliance/datasheets/{sheet_id}` | Get Datasheet |
| `POST` | `/api/v1/compliance/datasheets/{sheet_id}/publish` | Publish Datasheet |
| `GET` | `/api/v1/compliance/evidence` | List Compliance Evidence |
| `POST` | `/api/v1/compliance/evidence` | Upload Compliance Evidence |
| `POST` | `/api/v1/compliance/evidence/collect` | Collect Automated System Evidence |
| `GET` | `/api/v1/compliance/evidence/coverage-matrix` | Get Compliance Coverage Matrix |
| `GET` | `/api/v1/compliance/evidence/{evidence_id}` | Get Compliance Evidence |
| `PUT` | `/api/v1/compliance/evidence/{evidence_id}/approve` | Approve Compliance Evidence |
| `PATCH` | `/api/v1/compliance/evidence/{evidence_id}/coverage` | Manually Override Framework Coverage |
| `GET` | `/api/v1/compliance/frameworks` | List Frameworks |
| `PUT` | `/api/v1/compliance/frameworks/{framework}/adoption` | Set Framework Adoption |
| `POST` | `/api/v1/compliance/frameworks/{framework}/adoption-proposals` | Create Adoption Proposal |
| `POST` | `/api/v1/compliance/frameworks/{framework}/applicability-review` | Mark Applicability Reviewed |
| `PUT` | `/api/v1/compliance/frameworks/{framework}/applicability/{control_ref}` | Set Applicability |
| `GET` | `/api/v1/compliance/frameworks/{framework}/controls` | Get Framework Controls |
| `GET` | `/api/v1/compliance/frameworks/{framework}/custom-controls` | List Custom Controls |
| `POST` | `/api/v1/compliance/frameworks/{framework}/custom-controls` | Create Custom Control |
| `PATCH` | `/api/v1/compliance/frameworks/{framework}/custom-controls/{ref}` | Update Custom Control |
| `DELETE` | `/api/v1/compliance/frameworks/{framework}/custom-controls/{ref}` | Deactivate Custom Control |
| `GET` | `/api/v1/compliance/frameworks/{framework}/gaps` | Get Framework Gaps |
| `POST` | `/api/v1/compliance/gap-analysis` | Create Gap Analysis |
| `GET` | `/api/v1/compliance/gap-analysis/{analysis_id}` | Get Gap Analysis |
| `GET` | `/api/v1/compliance/iso42001/aims/objectives` | List AI Objectives |
| `POST` | `/api/v1/compliance/iso42001/aims/objectives` | Create AI Objective |
| `GET` | `/api/v1/compliance/iso42001/aims/policies` | List AIMS Policies |
| `POST` | `/api/v1/compliance/iso42001/aims/policies` | Create AIMS Policy |
| `PUT` | `/api/v1/compliance/iso42001/aims/policies/{policy_id}/approve` | Approve AIMS Policy |
| `POST` | `/api/v1/compliance/iso42001/assessment` | Create ISO 42001 Compliance Assessment |
| `POST` | `/api/v1/compliance/iso42001/continual-improvement` | Track Continual Improvement |
| `GET` | `/api/v1/compliance/iso42001/gaps` | Get Compliance Gaps |
| `POST` | `/api/v1/compliance/iso42001/lifecycle/impact-assessment/{model_id}` | Create AI Impact Assessment |
| `GET` | `/api/v1/compliance/iso42001/lifecycle/stage/{model_id}` | Get AI System Lifecycle Stage |
| `PUT` | `/api/v1/compliance/iso42001/lifecycle/stage/{model_id}/advance` | Advance AI System Lifecycle Stage |
| `POST` | `/api/v1/compliance/iso42001/management-review` | Create Management Review |
| `GET` | `/api/v1/compliance/iso42001/nist-mapping` | Get NIST AI RMF to ISO 42001 Mapping |
| `POST` | `/api/v1/compliance/iso42001/reports/certification` | Generate Certification Report |
| `GET` | `/api/v1/compliance/iso42001/score` | Get ISO 42001 Readiness Score |
| `POST` | `/api/v1/compliance/lineage/edges` | Create Lineage Edge |
| `GET` | `/api/v1/compliance/lineage/edges/{edge_id}` | Get Lineage Edge |
| `POST` | `/api/v1/compliance/lineage/edges/{edge_id}/execute` | Record Edge Execution |
| `POST` | `/api/v1/compliance/lineage/graph` | Get Lineage Graph |
| `POST` | `/api/v1/compliance/lineage/impact-analysis` | Create Impact Analysis |
| `GET` | `/api/v1/compliance/lineage/impact-analysis/{analysis_id}` | Get Impact Analysis |
| `GET` | `/api/v1/compliance/lineage/nodes` | List Lineage Nodes |
| `POST` | `/api/v1/compliance/lineage/nodes` | Create Lineage Node |
| `GET` | `/api/v1/compliance/lineage/nodes/{node_id}` | Get Lineage Node |
| `PUT` | `/api/v1/compliance/lineage/nodes/{node_id}` | Update Lineage Node |
| `GET` | `/api/v1/compliance/lineage/predictions/{prediction_id}` | Get Prediction Lineage |
| `POST` | `/api/v1/compliance/lineage/provenance` | Record Provenance |
| `GET` | `/api/v1/compliance/milestones` | List Milestones |
| `POST` | `/api/v1/compliance/milestones` | Create Milestone |
| `PATCH` | `/api/v1/compliance/milestones/{milestone_id}` | Update Milestone |
| `DELETE` | `/api/v1/compliance/milestones/{milestone_id}` | Delete Milestone |
| `GET` | `/api/v1/compliance/model-cards` | List Model Cards |
| `POST` | `/api/v1/compliance/model-cards` | Create Model Card |
| `POST` | `/api/v1/compliance/model-cards/refresh/{model_id}` | Refresh Model Card |
| `GET` | `/api/v1/compliance/model-cards/{card_id}` | Get Model Card |
| `PUT` | `/api/v1/compliance/model-cards/{card_id}` | Update Model Card |
| `DELETE` | `/api/v1/compliance/model-cards/{card_id}` | Delete Model Card |
| `POST` | `/api/v1/compliance/model-cards/{card_id}/publish` | Publish Model Card |

### Regulatory: EU AI Act and NIST AI RMF

| Method | Path | Summary |
| --- | --- | --- |
| `GET` | `/api/v1/regulatory/assessments` | Get Compliance Assessments |
| `POST` | `/api/v1/regulatory/assessments` | Create Compliance Assessment |
| `GET` | `/api/v1/regulatory/assessments/{assessment_id}` | Get Compliance Assessment |
| `PUT` | `/api/v1/regulatory/assessments/{assessment_id}` | Update Compliance Assessment |
| `GET` | `/api/v1/regulatory/compliance-summary` | Get Compliance Summary |
| `GET` | `/api/v1/regulatory/documentation` | Get Regulatory Documentation |
| `POST` | `/api/v1/regulatory/documentation` | Create Regulatory Documentation |
| `POST` | `/api/v1/regulatory/eu-ai-act/conformity-assessment` | Initiate Conformity Assessment |
| `GET` | `/api/v1/regulatory/eu-ai-act/conformity-assessments` | Get Conformity Assessments |
| `POST` | `/api/v1/regulatory/eu-ai-act/risk-classification` | Classify Eu Ai Act Risk |
| `POST` | `/api/v1/regulatory/nist-ai-rmf/assessment` | Assess Nist Ai Rmf |
| `GET` | `/api/v1/regulatory/risk-classifications` | Get Risk Classifications |
| `GET` | `/api/v1/regulatory/risk-classifications/high-risk` | Get High Risk Models |

### Data protection: GDPR and CCPA

| Method | Path | Summary |
| --- | --- | --- |
| `GET` | `/api/v1/data-protection/breach` | List Breach Notifications |
| `POST` | `/api/v1/data-protection/breach` | Report Breach |
| `PUT` | `/api/v1/data-protection/breach/{breach_id}/notify` | Process Breach Notifications |
| `POST` | `/api/v1/data-protection/ccpa/consumer-request` | Submit Ccpa Request |
| `GET` | `/api/v1/data-protection/ccpa/do-not-sell/{organization_id}/{consumer_email}` | Track Do Not Sell |
| `GET` | `/api/v1/data-protection/ccpa/metrics/{organization_id}` | Get Ccpa Metrics |
| `POST` | `/api/v1/data-protection/ccpa/opt-in` | Process Ccpa Opt In |
| `POST` | `/api/v1/data-protection/ccpa/opt-out` | Process Ccpa Opt Out |
| `GET` | `/api/v1/data-protection/ccpa/portal-compliance/{organization_id}` | Verify Consumer Portal |
| `GET` | `/api/v1/data-protection/ccpa/sale-disclosure/{organization_id}` | Get Ccpa Sale Disclosure |
| `GET` | `/api/v1/data-protection/consent` | List Consent Records |
| `POST` | `/api/v1/data-protection/gdpr/consent` | Create Consent Record |
| `POST` | `/api/v1/data-protection/gdpr/consent/{consent_id}/withdraw` | Withdraw Consent |
| `GET` | `/api/v1/data-protection/gdpr/data-minimization/{organization_id}` | Check Data Minimization |
| `GET` | `/api/v1/data-protection/gdpr/dsar` | List Gdpr Dsars |
| `POST` | `/api/v1/data-protection/gdpr/dsar` | Submit Gdpr Dsar |
| `PUT` | `/api/v1/data-protection/gdpr/dsar/{request_id}` | Update Gdpr Dsar |
| `GET` | `/api/v1/data-protection/gdpr/metrics/{organization_id}` | Get Gdpr Metrics |
| `GET` | `/api/v1/data-protection/gdpr/privacy-impact-assessment/{model_id}` | Conduct Privacy Impact Assessment |
| `POST` | `/api/v1/data-protection/gdpr/processing-record` | Document Processing Activity |
| `POST` | `/api/v1/data-protection/gdpr/retention-policy` | Create Retention Policy |

### Audit trail

| Method | Path | Summary |
| --- | --- | --- |
| `POST` | `/api/v1/audit-trail` | Create Audit Log |
| `GET` | `/api/v1/audit-trail/activity/users/{user_id}` | Get User Activity |
| `POST` | `/api/v1/audit-trail/compliance/data-subject-request` | Process Data Subject Request |
| `POST` | `/api/v1/audit-trail/compliance/query` | Query Compliance Events |
| `GET` | `/api/v1/audit-trail/compliance/report` | Get Compliance Report |
| `POST` | `/api/v1/audit-trail/export` | Request Export |
| `GET` | `/api/v1/audit-trail/export/{export_id}` | Get Export Status |
| `GET` | `/api/v1/audit-trail/export/{export_id}/download` | Download Export |
| `POST` | `/api/v1/audit-trail/query` | Query Audit Logs |
| `GET` | `/api/v1/audit-trail/retention-policies` | Get Retention Policies |
| `POST` | `/api/v1/audit-trail/retention-policies` | Create Retention Policy |
| `POST` | `/api/v1/audit-trail/retention-policies/apply` | Apply Retention Policies |
| `PUT` | `/api/v1/audit-trail/retention-policies/{policy_id}` | Update Retention Policy |
| `GET` | `/api/v1/audit-trail/security/summary` | Get Security Summary |
| `GET` | `/api/v1/audit-trail/statistics` | Get Statistics |
| `GET` | `/api/v1/audit-trail/timeline` | Get Timeline |
| `POST` | `/api/v1/audit-trail/verify-integrity` | Verify Integrity |
| `GET` | `/api/v1/audit-trail/{log_id}` | Get Audit Log |

### Report types

| Method | Path | Summary |
| --- | --- | --- |
| `GET` | `/api/v1/report-brand-profiles` | List Brand Profiles |
| `POST` | `/api/v1/report-brand-profiles` | Create Brand Profile |
| `GET` | `/api/v1/report-brand-profiles/{profile_id}` | Get Brand Profile |
| `PATCH` | `/api/v1/report-brand-profiles/{profile_id}` | Update Brand Profile |
| `DELETE` | `/api/v1/report-brand-profiles/{profile_id}` | Delete Brand Profile |
| `GET` | `/api/v1/report-brand-profiles/{profile_id}/logo` | Get Brand Profile Logo |
| `POST` | `/api/v1/report-brand-profiles/{profile_id}/logo` | Upload Brand Profile Logo |
| `GET` | `/api/v1/report-studio/templates` | List Studio Templates |
| `POST` | `/api/v1/report-studio/templates` | Create Studio Template |
| `GET` | `/api/v1/report-studio/templates/{template_id}` | Get Studio Template |
| `PATCH` | `/api/v1/report-studio/templates/{template_id}` | Update Studio Template |
| `DELETE` | `/api/v1/report-studio/templates/{template_id}` | Delete Studio Template |
| `GET` | `/api/v1/report-types` | List Report Types |
| `POST` | `/api/v1/report-types/{report_type_id}/generate` | Generate Report |
| `POST` | `/api/v1/report-types/{report_type_id}/preview` | Preview Report |

### Reports and exports

| Method | Path | Summary |
| --- | --- | --- |
| `GET` | `/api/v1/export/configs` | List Export Configs |
| `POST` | `/api/v1/export/configs` | Create Export Config |
| `GET` | `/api/v1/export/configs/{config_id}` | Get Export Config |
| `PUT` | `/api/v1/export/configs/{config_id}` | Update Export Config |
| `DELETE` | `/api/v1/export/configs/{config_id}` | Delete Export Config |
| `POST` | `/api/v1/export/evidence-pack` | Create Evidence Pack |
| `GET` | `/api/v1/export/exports` | List Exports |
| `POST` | `/api/v1/export/exports` | Create Export |
| `POST` | `/api/v1/export/exports/bulk` | Create Bulk Export |
| `POST` | `/api/v1/export/exports/verify` | Verify Export |
| `GET` | `/api/v1/export/exports/{export_id}` | Get Export |
| `GET` | `/api/v1/export/exports/{export_id}/download` | Download Export |
| `POST` | `/api/v1/export/exports/{export_id}/share` | Create Export Share |
| `GET` | `/api/v1/export/exports/{export_id}/shares` | List Export Shares |
| `GET` | `/api/v1/export/exports/{export_id}/signoffs` | List Export Signoffs |
| `POST` | `/api/v1/export/exports/{export_id}/signoffs` | Create Export Signoff |
| `GET` | `/api/v1/export/integrations` | List Integrations |
| `POST` | `/api/v1/export/integrations` | Create Integration |
| `GET` | `/api/v1/export/integrations/{integration_id}` | Get Integration |
| `PUT` | `/api/v1/export/integrations/{integration_id}` | Update Integration |
| `DELETE` | `/api/v1/export/integrations/{integration_id}` | Delete Integration |
| `GET` | `/api/v1/export/integrations/{integration_id}/logs` | List Integration Logs |
| `GET` | `/api/v1/export/integrations/{integration_id}/stats` | Get Integration Stats |
| `POST` | `/api/v1/export/integrations/{integration_id}/test` | Test Integration |
| `GET` | `/api/v1/export/scheduled-reports` | List Scheduled Reports |
| `POST` | `/api/v1/export/scheduled-reports` | Create Scheduled Report |
| `GET` | `/api/v1/export/scheduled-reports/{report_id}` | Get Scheduled Report |
| `PUT` | `/api/v1/export/scheduled-reports/{report_id}` | Update Scheduled Report |
| `DELETE` | `/api/v1/export/scheduled-reports/{report_id}` | Delete Scheduled Report |
| `GET` | `/api/v1/export/scheduled-reports/{report_id}/deliveries` | List Deliveries |
| `POST` | `/api/v1/export/scheduled-reports/{report_id}/run` | Run Scheduled Report |
| `GET` | `/api/v1/export/share-presets` | List Share Presets |
| `GET` | `/api/v1/export/shares/{share_slug}/download` | Download Export Share |
| `POST` | `/api/v1/export/shares/{share_slug}/request-otp` | Request Export Share Otp |
| `POST` | `/api/v1/export/shares/{share_slug}/revoke` | Revoke Export Share |
| `POST` | `/api/v1/export/shares/{share_slug}/verify-otp` | Verify Export Share Otp |
| `GET` | `/api/v1/export/templates` | List Report Templates |
| `POST` | `/api/v1/export/templates` | Create Report Template |
| `GET` | `/api/v1/export/templates/{template_id}` | Get Report Template |
| `PUT` | `/api/v1/export/templates/{template_id}` | Update Report Template |
| `DELETE` | `/api/v1/export/templates/{template_id}` | Delete Report Template |

### GitHub and model diff

| Method | Path | Summary |
| --- | --- | --- |
| `POST` | `/api/v1/github/automation/repositories/{repository_id}/branch-protection/enable` | Enable Branch Protection |
| `GET` | `/api/v1/github/automation/repositories/{repository_id}/branch-protection/{branch}` | Get Branch Protection |
| `POST` | `/api/v1/github/automation/repositories/{repository_id}/branch-protection/{branch}/required-checks` | Add Required Status Check |
| `DELETE` | `/api/v1/github/automation/repositories/{repository_id}/branch-protection/{branch}/required-checks` | Remove Required Status Check |
| `POST` | `/api/v1/github/automation/repositories/{repository_id}/checks/comparison` | Create Comparison Check |
| `POST` | `/api/v1/github/automation/repositories/{repository_id}/checks/validation` | Create Validation Check |
| `PUT` | `/api/v1/github/automation/repositories/{repository_id}/pr-comments/{pr_number}` | Upsert Comment |
| `POST` | `/api/v1/github/automation/repositories/{repository_id}/pr-comments/{pr_number}/comparison` | Post Comparison Comment |
| `POST` | `/api/v1/github/automation/repositories/{repository_id}/pr-comments/{pr_number}/validation` | Post Validation Comment |
| `POST` | `/api/v1/github/connect/pat` | Connect With Pat |
| `GET` | `/api/v1/github/connections` | List Connections |
| `DELETE` | `/api/v1/github/connections/{connection_id}` | Delete Connection |
| `GET` | `/api/v1/github/connections/{connection_id}/repositories` | List Repositories |
| `POST` | `/api/v1/github/connections/{connection_id}/verify` | Verify Connection |
| `GET` | `/api/v1/github/diff/architecture-changes/{model_id}` | Get Architecture Changes Timeline |
| `GET` | `/api/v1/github/diff/changelog` | Generate Changelog |
| `POST` | `/api/v1/github/diff/compare` | Compare Model Versions |
| `GET` | `/api/v1/github/diff/model/{model_id}/versions` | Get Model Version History |
| `GET` | `/api/v1/github/oauth/authorize` | Github Oauth Authorize |
| `GET` | `/api/v1/github/repositories` | List Tracked Repositories |
| `POST` | `/api/v1/github/repositories/track` | Track Repository |
| `DELETE` | `/api/v1/github/repositories/{repository_id}` | Untrack Repository |
| `POST` | `/api/v1/github/repositories/{repository_id}/webhook` | Setup Webhook |

### n8n webhooks

| Method | Path | Summary |
| --- | --- | --- |
| `GET` | `/api/v1/webhooks/n8n/config` | Get N8N Webhook Config |
| `POST` | `/api/v1/webhooks/n8n/execution` | Receive N8N Execution |
| `POST` | `/api/v1/webhooks/n8n/test` | Test N8N Webhook |

### LiteLLM gateway

The ingest route is called by your LiteLLM proxy with a `wbx_lgw_` gateway token, not an API key. The
other routes need an organisation administrator; rotating the token also needs a signed-in session, not an
API key. For LiteLLM, `{provider}` is `litellm`. The older `/api/v1/integrations/litellm/...` paths still
work and behave the same. See [LiteLLM gateway](../integrations/litellm.md).

| Method | Path | Summary |
| --- | --- | --- |
| `POST` | `/api/v1/llm/gateways/litellm/ingest` | Ingest Litellm |
| `GET` | `/api/v1/integrations/llm-gateways` | List Gateway Connections |
| `GET` | `/api/v1/integrations/llm-gateways/{provider}` | Get Gateway Connection |
| `POST` | `/api/v1/integrations/llm-gateways/{provider}` | Connect Gateway |
| `PATCH` | `/api/v1/integrations/llm-gateways/{provider}` | Update Gateway Connection |
| `DELETE` | `/api/v1/integrations/llm-gateways/{provider}` | Delete Gateway Connection |
| `POST` | `/api/v1/integrations/llm-gateways/{provider}/token/rotate` | Rotate Gateway Token |
| `GET` | `/api/v1/integrations/llm-gateways/{provider}/rules` | List Gateway Rules |
| `POST` | `/api/v1/integrations/llm-gateways/{provider}/rules` | Create Gateway Rule |
| `DELETE` | `/api/v1/integrations/llm-gateways/{provider}/rules/{rule_id}` | Delete Gateway Rule |
| `GET` | `/api/v1/integrations/llm-gateways/{provider}/unattributed` | Get Gateway Unattributed |
| `GET` | `/api/v1/integrations/llm-gateways/{provider}/config-snippet` | Get Gateway Config Snippet |

### API keys and two-factor authentication

| Method | Path | Summary |
| --- | --- | --- |
| `GET` | `/api/v1/api-keys` | List Api Keys |
| `POST` | `/api/v1/api-keys` | Create Api Key |
| `DELETE` | `/api/v1/api-keys/{key_id}` | Revoke Api Key Endpoint |
| `POST` | `/api/v1/two-factor/disable` | Disable Two Factor |
| `POST` | `/api/v1/two-factor/enable` | Enable Two Factor |
| `POST` | `/api/v1/two-factor/regenerate-backup-codes` | Regenerate Backup Codes |
| `POST` | `/api/v1/two-factor/setup` | Setup Two Factor |
| `GET` | `/api/v1/two-factor/status` | Get Two Factor Status |

### Users and organization

| Method | Path | Summary |
| --- | --- | --- |
| `GET` | `/api/v1/organizations/me/subscription` | Get My Subscription |
| `GET` | `/api/v1/organizations/{organization_id}/branding` | Get Organization Branding |
| `PATCH` | `/api/v1/organizations/{organization_id}/branding` | Update Organization Branding |
| `GET` | `/api/v1/organizations/{organization_id}/branding/logo` | Get Organization Branding Logo |
| `POST` | `/api/v1/organizations/{organization_id}/branding/logo` | Upload Organization Logo |
| `GET` | `/api/v1/organizations/{organization_id}/settings` | Get Organization Settings |
| `PATCH` | `/api/v1/organizations/{organization_id}/settings` | Update Organization Settings |
| `GET` | `/api/v1/users` | List Users |
| `POST` | `/api/v1/users` | Create User |
| `GET` | `/api/v1/users/me` | Get Current User Info |
| `PATCH` | `/api/v1/users/me` | Update Current User Info |
| `POST` | `/api/v1/users/me/avatar` | Upload Current User Avatar |
| `POST` | `/api/v1/users/me/deletion` | Delete My Account |
| `GET` | `/api/v1/users/me/export` | Export My Data |
| `POST` | `/api/v1/users/me/password` | Change Current User Password |
| `GET` | `/api/v1/users/me/preferences` | Get My Preferences |
| `PUT` | `/api/v1/users/me/preferences` | Update My Preferences |
| `GET` | `/api/v1/users/{user_id}` | Get User |
| `PATCH` | `/api/v1/users/{user_id}` | Update User |
| `DELETE` | `/api/v1/users/{user_id}` | Delete User |

### Integrations

| Method | Path | Summary |
| --- | --- | --- |
| `GET` | `/api/v1/integrations/status` | Get Integrations Status |

### Dashboard data

| Method | Path | Summary |
| --- | --- | --- |
| `GET` | `/api/v1/dashboard/health` | Get System Health |
| `POST` | `/api/v1/dashboard/metrics/query` | Query Metrics |
| `GET` | `/api/v1/dashboard/metrics/recent` | Get Recent Metrics |
| `GET` | `/api/v1/dashboard/models/status` | Get Models Status |
| `GET` | `/api/v1/dashboard/models/{model_id}/activity` | Get Model Activity |
| `GET` | `/api/v1/dashboard/stats` | Get Dashboard Stats |

## Related

- [Python SDK](index.md)
- [API Keys](../account/api-keys.md)
- [Governance Events](../get-started/governance-events.md)
- [Policy-as-Code & GitOps](../integrations/policy-as-code.md)
- [MCP Server](../integrations/mcp.md)
