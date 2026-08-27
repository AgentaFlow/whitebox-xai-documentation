# MCP Server

[MCP (Model Context Protocol)](https://modelcontextprotocol.io/) is an open protocol for
connecting AI clients — Claude Desktop, Claude Code, LangChain agents, custom agent
harnesses — to external tools and data.

WhiteBoxXAI ships an MCP server so any MCP-compatible client can register models, log
predictions, run drift detection, run bias/fairness audits, generate SHAP/LIME and LLM
explanations, query the AI Risk Register and governance RACI grid, monitor LLM/RAG calls and
content safety, manage alert rules, and trace multi-agent workflows — without writing
integration code. It's the recommended path for non-Python clients and for agents that need to
reach WhiteBoxXAI as a tool. If you're writing Python, use the [SDK](/sdk/) directly
instead.

Every tool calls the real WhiteBoxXAI API through the SDK — with one documented exception, see
[Current limitations](#current-limitations), none return mocked or placeholder data.

## Prerequisites

- A WhiteBoxXAI account.
- Python 3.11+ (for the pip install) or Docker.
- An [API key](/account/api-keys/) — see [Authentication](#authentication).

## Install

```bash
pip install whiteboxxai-mcp
```

## Quick start

```bash
export WHITEBOXXAI_MCP_API_BASE_URL="https://api.whiteboxxai.com"
export WHITEBOXXAI_MCP_API_KEY="wbx_live_..."
whiteboxxai-mcp
```

This runs the server over **stdio** — your MCP client spawns the `whiteboxxai-mcp` process and
talks to its stdin/stdout. There's also a hosted **Streamable HTTP** endpoint if you'd rather
not run your own process — see [Connect over HTTP](#connect-over-http).

## Authentication

For stdio, configure exactly one of these modes:

| Mode | Environment variables | Notes |
| --- | --- | --- |
| **API key** (recommended) | `WHITEBOXXAI_MCP_API_KEY` | Scoped and revocable. Doesn't expire on a schedule — only when you revoke it. |
| Service account | `WHITEBOXXAI_MCP_EMAIL` + `WHITEBOXXAI_MCP_PASSWORD` | Refreshes its own session automatically. The account **must have 2FA disabled** — the server can't complete a headless 2FA challenge. |
| Static JWT | `WHITEBOXXAI_MCP_JWT` | A pre-issued login token. Cannot be refreshed; tool calls start failing once it expires (~30 minutes) until you restart with a fresh token. |

Connection settings:

| Variable | Default |
| --- | --- |
| `WHITEBOXXAI_MCP_API_BASE_URL` | `https://api.whiteboxxai.com` |
| `WHITEBOXXAI_MCP_API_TIMEOUT` | `30` (seconds) |

Use an API key for anything long-running. It has a much smaller blast radius than a
password: a leaked key is revoked with a single call, while a leaked password means a full
credential rotation. See [API Keys](/account/api-keys/) for how to issue one.

## Connect from a client

### Claude Desktop and Claude Code

Add the server to your client's MCP configuration — Claude Desktop's
`claude_desktop_config.json`, or a project-level `.mcp.json`:

```json
{
  "mcpServers": {
    "whiteboxxai": {
      "command": "whiteboxxai-mcp",
      "env": {
        "WHITEBOXXAI_MCP_API_BASE_URL": "https://api.whiteboxxai.com",
        "WHITEBOXXAI_MCP_API_KEY": "wbx_live_..."
      }
    }
  }
}
```

### Docker

If you'd rather not install into your client's Python environment, point `command` at
`docker`:

```json
{
  "mcpServers": {
    "whiteboxxai": {
      "command": "docker",
      "args": ["run", "--rm", "-i", "-e", "WHITEBOXXAI_MCP_API_KEY", "whiteboxxai-mcp"]
    }
  }
}
```

### Other clients

Any MCP client library with stdio transport can spawn `whiteboxxai-mcp` the same way.
Consult that client's own MCP integration docs for its configuration format.

## Connect over HTTP

The production backend also serves the MCP server directly at `https://api.whiteboxxai.com/mcp`
over Streamable HTTP, for clients that would rather not spawn and manage a local process.

```json
{
  "mcpServers": {
    "whiteboxxai": {
      "url": "https://api.whiteboxxai.com/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_JWT_OR_API_KEY"
      }
    }
  }
}
```

Exact config shape depends on your client — check its docs for how it expresses an HTTP MCP
server with a custom header.

HTTP mode's authentication model is different from stdio's, because one process now serves
every organization instead of one fixed credential per process:

- Every request must carry its own `Authorization: Bearer <token>` header — a JWT from
  `POST /api/v1/auth/login`, or an [API key](/account/api-keys/). There's no shared or
  default credential, and no fallback to the `WHITEBOXXAI_MCP_*` environment variables stdio
  mode uses. A request without a valid header is rejected before an MCP session is even
  created.
- A JWT-authenticated HTTP session **does not refresh itself** — the client library minted
  it, not the server, so the server has nothing to refresh it with. Expect to reconnect with
  a fresh JWT roughly every 30 minutes, or use a long-lived API key instead to avoid that
  entirely.

## Tool reference

172 tools across sixteen domains, all prefixed `whitebox_`. Every tool calls the real backend
through the SDK — with one documented exception, [noted below](#current-limitations), none
return mocked or placeholder data.

### Models

| Tool | Description |
| --- | --- |
| `whitebox_models_register` | Register a new model in the model registry. |
| `whitebox_models_get` | Get a registered model's full record by `model_id`. |
| `whitebox_models_list` | List registered models, optionally filtered by status, type, tags, or search text. |
| `whitebox_models_update` | Partially update a model's metadata. |
| `whitebox_models_update_status` | Set a model's status: `ACTIVE`, `INACTIVE`, `DEPRECATED`, or `ARCHIVED`. |
| `whitebox_models_versions` | List all versions of a model by name, newest first. |
| `whitebox_models_latest` | Get the most recently created version of a model by name. |
| `whitebox_models_update_baseline` | Refresh a model's baseline metrics/profile, e.g. after retraining. |
| `whitebox_models_archive` | Archive a model (soft delete, restorable). |
| `whitebox_models_restore` | Restore a previously archived model to active status. |
| `whitebox_models_delete` | Permanently delete a model. Cannot be undone — prefer archive. |

### Predictions

| Tool | Description |
| --- | --- |
| `whitebox_predictions_log` | Log a single model prediction for monitoring. |
| `whitebox_predictions_log_batch` | Log up to 1,000 predictions in one call. |
| `whitebox_predictions_get` | Get a single logged prediction by `prediction_id`. |
| `whitebox_predictions_query` | Query logged predictions for a model, optionally within a time range. |
| `whitebox_predictions_stats` | Get prediction volume and latency statistics for a model. |
| `whitebox_predictions_recent` | Get a model's most recent predictions (max 100). |

### Drift

| Tool | Description |
| --- | --- |
| `whitebox_drift_detect` | Run an ad-hoc data drift check (not persisted). |
| `whitebox_drift_create_report` | Run drift analysis and persist the result as a drift report. |
| `whitebox_drift_list_reports` | List drift reports for a model, most recent first. |
| `whitebox_drift_get_report` | Get a specific drift report, including per-feature statistics. |
| `whitebox_drift_trend` | Get a model's drift trend over time (1–90 days). |

### Fairness

| Tool | Description |
| --- | --- |
| `whitebox_fairness_audit` | Run a bias/fairness audit across demographic groups. |
| `whitebox_fairness_get_audit` | Get full bias audit results by `audit_id`. |
| `whitebox_fairness_list_audits` | List bias audit summaries, optionally filtered to one model. |
| `whitebox_fairness_bias_history` | Get a model's historical bias/fairness trend (1–365 days). |
| `whitebox_fairness_metric_history` | Get history for one specific fairness metric on a model. |
| `whitebox_fairness_latest_audit` | Get the most recent bias audit for a model. |

!!! tip "Drift and fairness tools have side effects"
    `whitebox_drift_create_report` and `whitebox_fairness_audit` persist records — and a
    `high` or `critical` result will auto-draft an entry in your [AI Risk
    Register](/user-guide/risk-register/). That's usually what you want, but it's worth
    knowing before you let an agent call them in a loop.

### AI Risk Register

Read-only — entries are populated from the dashboard or auto-drafted server-side (see the tip
above), not written by MCP callers.

| Tool | Description |
| --- | --- |
| `whitebox_risk_register_list` | List risk register entries, optionally filtered by status, severity, category, or model. Each item includes its ISO 42001 clause references. |
| `whitebox_risk_register_get` | Get a single risk register entry by ID. |
| `whitebox_risk_register_portfolio` | Get the likelihood × impact heat map and top open risks across the organization. |

### Governance / RACI

Read-only — creating review requests, voting, and RACI assignment stay human/dashboard
workflows.

| Tool | Description |
| --- | --- |
| `whitebox_governance_list_boards` | List governance review boards for the organization. |
| `whitebox_governance_list_review_requests` | List review requests, optionally filtered by board or status. |
| `whitebox_governance_raci_grid` | Get the RACI grid (Responsible/Accountable/Consulted/Informed) for review requests, optionally scoped to one board. |

### Explanations

The async two-tool pattern — start the job, then poll — proxying the real Celery-backed
SHAP/LIME pipeline.

| Tool | Description |
| --- | --- |
| `whitebox_explanations_generate_async` | Start SHAP/LIME explanation generation for a model instance. Returns immediately with a `pending` `explanation_id` rather than blocking until it's ready. |
| `whitebox_explanations_get` | Get an explanation by ID — status, and once complete, `feature_weights`, `base_value`, and `score`. Poll after `generate_async` until status is no longer `pending`. |

The synchronous `/generate` endpoint and the bulk/compare/visualize/config-CRUD explanation
endpoints aren't wrapped as tools yet — use the [SDK](/sdk/) or [REST
API](/sdk/api-reference/#explainability-xai) for those.

### LLM Explainability

Attention/token-importance/sensitivity/counterfactual analysis and prompt debugging for LLM
calls — distinct from the model-level SHAP/LIME tools above.

| Tool | Description |
| --- | --- |
| `whitebox_llm_xai_attention` | Analyze transformer attention patterns for a prompt/completion pair. |
| `whitebox_llm_xai_token_importance` | Calculate token-level importance scores (gradient/attention/perturbation-based). |
| `whitebox_llm_xai_prompt_sensitivity` | Test prompt robustness by perturbing it and measuring output stability. |
| `whitebox_llm_xai_counterfactuals` | Generate minimal prompt edits that change the model's output. |
| `whitebox_llm_xai_debug_prompt` | Assess prompt quality across multiple dimensions with improvement recommendations. |
| `whitebox_llm_xai_get_explanation` | Get a specific attention/token-importance/sensitivity/counterfactual explanation by ID. |
| `whitebox_llm_xai_get_prompt_analysis` | Get a specific prompt debug analysis by ID. |
| `whitebox_llm_xai_list_explanations` | List explanations, optionally filtered by LLM log, type, or model. |
| `whitebox_llm_xai_list_prompt_analyses` | List prompt analyses, optionally filtered by similar prompt text, model, or quality score. |
| `whitebox_llm_xai_stats` | Get aggregated LLM-XAI usage statistics. |
| `whitebox_llm_xai_visualize_attention` | Get attention-heatmap visualization data for an explanation. |
| `whitebox_llm_xai_visualize_token_importance` | Get token-importance bar-chart visualization data for an explanation. |
| `whitebox_llm_xai_batch_analyze` | Run multiple explanation analyses on a single LLM log in one call. |

### LLM Monitoring

Token/cost/latency tracking for LLM calls.

| Tool | Description |
| --- | --- |
| `whitebox_llm_monitoring_log_call` | Log a single LLM call with real token/cost/latency data. |
| `whitebox_llm_monitoring_log_calls_batch` | Log multiple LLM calls in one request (max 1,000). |
| `whitebox_llm_monitoring_get_log` | Get a single LLM log by ID. |
| `whitebox_llm_monitoring_query_logs` | Query LLM logs with filters and pagination. |
| `whitebox_llm_monitoring_session_logs` | Get all LLM logs for a specific session. |
| `whitebox_llm_monitoring_usage_stats` | Get aggregated LLM usage statistics (tokens, cost, latency, by model). |
| `whitebox_llm_monitoring_recent` | Get recent LLM logs for a model. |
| `whitebox_llm_monitoring_cost_breakdown` | Get detailed cost breakdown (platform-wide aggregate). |
| `whitebox_llm_monitoring_performance` | Get LLM latency/throughput performance metrics. |
| `whitebox_llm_monitoring_trends_tokens` | Get token usage trend over time. |
| `whitebox_llm_monitoring_trends_costs` | Get cost trend over time. |
| `whitebox_llm_monitoring_usage_stats_detailed` | Get detailed, grouped usage statistics. |
| `whitebox_llm_monitoring_cost_threshold_alert` | Check whether recent costs exceed a threshold. |
| `whitebox_llm_monitoring_latency_threshold_alert` | Check whether recent average latency exceeds a threshold. |
| `whitebox_llm_monitoring_error_rate_alert` | Check whether the recent error rate exceeds a threshold. |
| `whitebox_llm_monitoring_cleanup_logs` | Delete LLM logs older than N days. Admin-only on the backend. |

### RAG Monitoring

| Tool | Description |
| --- | --- |
| `whitebox_rag_log_retrieval` | Log a retrieval operation, with real quality metrics computed server-side when ground truth is supplied. |
| `whitebox_rag_get_retrieval` | Get a specific retrieval log by ID. |
| `whitebox_rag_list_retrievals` | Query retrieval logs for a model. |
| `whitebox_rag_stats` | Get aggregated RAG statistics (precision/recall/MRR/NDCG/relevance/latency). |
| `whitebox_rag_trends` | Get RAG quality metrics trend over time. |
| `whitebox_rag_create_evaluation` | Create a RAG evaluation, aggregating already-logged retrievals for each test query. |
| `whitebox_rag_get_evaluation` | Get a specific RAG evaluation by ID. |
| `whitebox_rag_list_evaluations` | List RAG evaluations with optional filters. |
| `whitebox_rag_metrics_precision` | Get Precision@K metrics for given K values. |
| `whitebox_rag_metrics_relevance` | Get average relevance metrics (context/answer/faithfulness). |

### Content Safety

| Tool | Description |
| --- | --- |
| `whitebox_safety_analyze` | Analyze content for toxicity, PII, and harmful content. |
| `whitebox_safety_analyze_batch` | Analyze multiple content items in one request (max 100). |
| `whitebox_safety_get_score` | Get a specific safety score by ID. |
| `whitebox_safety_get_scores` | Query safety analysis scores with filters. |
| `whitebox_safety_stats` | Get aggregated safety statistics. |
| `whitebox_safety_trends` | Get content-safety trend over time. |
| `whitebox_safety_create_threshold` | Create a safety threshold configuration controlling when content is flagged unsafe. |
| `whitebox_safety_list_thresholds` | List safety thresholds for the organization. |
| `whitebox_safety_get_threshold` | Get a specific safety threshold by ID. |
| `whitebox_safety_update_threshold` | Update a safety threshold configuration. |
| `whitebox_safety_delete_threshold` | Delete a safety threshold configuration. |

### Alerts

Core rule/instance CRUD.

| Tool | Description |
| --- | --- |
| `whitebox_alerts_create_rule` | Create an alert rule with threshold or anomaly-based conditions. |
| `whitebox_alerts_list_rules` | List alert rules for the organization. |
| `whitebox_alerts_get_rule` | Get a specific alert rule by ID. |
| `whitebox_alerts_update_rule` | Update an alert rule. |
| `whitebox_alerts_delete_rule` | Delete an alert rule. |
| `whitebox_alerts_evaluate_rule` | Test whether a rule would trigger given metric values, without creating a real alert. |
| `whitebox_alerts_list_instances` | List triggered alert instances with optional filters. |
| `whitebox_alerts_get_instance` | Get a specific triggered alert instance by ID. |
| `whitebox_alerts_acknowledge` | Acknowledge an active alert instance. |
| `whitebox_alerts_resolve` | Resolve an alert instance. |
| `whitebox_alerts_snooze` | Snooze an active alert instance. |
| `whitebox_alerts_statistics` | Get alert statistics (counts, severity distribution, MTTA/MTTR). |

### Alert Intelligence

Correlation, root-cause analysis, priority scoring, pattern detection, and fatigue reduction
across triggered alerts.

| Tool | Description |
| --- | --- |
| `whitebox_alert_intelligence_detect_correlations` | Find alerts likely correlated with a given alert within a lookback window. |
| `whitebox_alert_intelligence_get_correlation_clusters` | Get clustered groups of correlated alerts for the organization. |
| `whitebox_alert_intelligence_analyze_root_cause` | Run root-cause analysis for an alert. |
| `whitebox_alert_intelligence_verify_root_cause` | Record a human verification (confirmed/rejected) of a root-cause finding. |
| `whitebox_alert_intelligence_get_similar_incidents` | Find past incidents with a similar root cause. |
| `whitebox_alert_intelligence_calculate_priority` | Calculate a priority score for an alert. |
| `whitebox_alert_intelligence_adjust_priority` | Manually override an alert's calculated priority. |
| `whitebox_alert_intelligence_get_priority_distribution` | Get the distribution of alert priorities over a time window. |
| `whitebox_alert_intelligence_get_priority_queue` | Get the current priority-ordered alert queue. |
| `whitebox_alert_intelligence_detect_patterns` | Detect recurring alert patterns over a time window. |
| `whitebox_alert_intelligence_get_pattern` | Get a specific detected pattern by ID. |
| `whitebox_alert_intelligence_deactivate_pattern` | Deactivate a detected pattern. |
| `whitebox_alert_intelligence_apply_fatigue_reduction` | Apply alert-fatigue reduction rules for the organization. |
| `whitebox_alert_intelligence_get_throttle_recommendations` | Get recommended throttling rules to reduce alert noise. |
| `whitebox_alert_intelligence_ml_grouping` | Group related alerts using ML-based clustering. |
| `whitebox_alert_intelligence_temporal_grouping` | Group alerts that fired close together in time. |
| `whitebox_alert_intelligence_predict_trends` | Predict alert volume trend direction. |
| `whitebox_alert_intelligence_detect_seasonality` | Detect seasonal/cyclical patterns in alert volume. |
| `whitebox_alert_intelligence_enhanced_trend_analysis` | Get an enhanced trend analysis for an alert metric. |
| `whitebox_alert_intelligence_forecast_volume` | Forecast future alert volume. |

!!! note "Four forecasting tools return placeholder data"
    `predict_trends`, `detect_seasonality`, `enhanced_trend_analysis`, and `forecast_volume`
    are wired end-to-end but their underlying forecasting models aren't implemented yet — they
    currently return hardcoded placeholder values rather than a real prediction. Every other
    tool on this page, including the rest of this domain, calls real computed logic.

### Alert Management

Higher-level, agent-facing operations on top of the core Alerts domain: history, analytics,
fatigue detection, and comments.

| Tool | Description |
| --- | --- |
| `whitebox_alert_management_get_history` | Get the full action timeline for an alert (triggered/acknowledged/snoozed/resolved). |
| `whitebox_alert_management_get_analytics` | Get aggregated alert analytics over a time range. |
| `whitebox_alert_management_get_trend` | Get alert volume/severity trend over a time range. |
| `whitebox_alert_management_get_velocity` | Get alert generation rate trends (hour-over-hour, day-over-day). |
| `whitebox_alert_management_get_grouping` | Get alerts grouped by a given dimension. |
| `whitebox_alert_management_detect_fatigue` | Detect alert-fatigue indicators and get recommendations. |
| `whitebox_alert_management_acknowledge` | Acknowledge an alert, with optional notes. |
| `whitebox_alert_management_resolve` | Resolve an alert, with optional resolution notes. |
| `whitebox_alert_management_snooze` | Snooze an alert for a given number of minutes. |
| `whitebox_alert_management_create_comment` | Add a comment to an alert. |
| `whitebox_alert_management_get_comments` | Get an alert's comment thread. |
| `whitebox_alert_management_process_pending_escalations` | Manually trigger processing of pending escalations (normally run on a schedule). |
| `whitebox_alert_management_unsnooze_expired_alerts` | Manually trigger reactivation of alerts whose snooze period has ended (normally run on a schedule). |

### Multi-Agent Workflows

CrewAI/LangChain/n8n workflow, agent, execution, interaction, and task tracking — see [Multi-Agent
Workflow Monitoring](/sdk/multi-agent-monitoring/).

| Tool | Description |
| --- | --- |
| `whitebox_agent_workflows_create_and_start` | Create a new multi-agent workflow. |
| `whitebox_agent_workflows_start` | Mark a workflow as running. |
| `whitebox_agent_workflows_complete` | Mark a workflow completed/failed/cancelled and optionally trigger analytics. |
| `whitebox_agent_workflows_list` | List workflows for the organization. |
| `whitebox_agent_workflows_get` | Get a workflow by ID. |
| `whitebox_agent_workflows_delete` | Delete a workflow and all related data. |
| `whitebox_agent_workflows_register_agent` | Register an agent in a workflow. |
| `whitebox_agent_workflows_list_agents` | List all agents registered in a workflow. |
| `whitebox_agent_workflows_create_execution` | Log that an agent started executing. |
| `whitebox_agent_workflows_log_interaction` | Log agent-to-agent communication (delegation/handoff/query/feedback/broadcast). |
| `whitebox_agent_workflows_list_interactions` | List all agent-to-agent interactions in a workflow. |
| `whitebox_agent_workflows_create_task` | Create a workflow task. |
| `whitebox_agent_workflows_update_task_status` | Update a task's status. |
| `whitebox_agent_workflows_list_tasks` | List all tasks in a workflow. |
| `whitebox_agent_workflows_analytics` | Get workflow analytics (tokens, costs, execution counts, durations). |
| `whitebox_agent_workflows_cost_breakdown` | Get per-agent cost breakdown. Can take up to ~30s. |
| `whitebox_agent_workflows_bottlenecks` | Identify workflow bottlenecks. Can take up to ~30s. |
| `whitebox_agent_workflows_timeline` | Get a chronological timeline of workflow events. Can take up to ~30s. |

### Metrics

| Tool | Description |
| --- | --- |
| `whitebox_metrics_create` | Create a new metric record for a model. |
| `whitebox_metrics_create_batch` | Create multiple metrics at once (max 100). |
| `whitebox_metrics_calculate_classification` | Calculate classification metrics from predictions, without persisting anything. |
| `whitebox_metrics_calculate_regression` | Calculate regression metrics from predictions, without persisting anything. |
| `whitebox_metrics_get_model_metrics` | Get metrics recorded for a model, with optional filters. |
| `whitebox_metrics_latest` | Get the most recent metric of a specific type for a model. |
| `whitebox_metrics_timeseries` | Get time series data for a metric over a date range. |
| `whitebox_metrics_aggregate` | Get metrics aggregated over a period (daily/weekly/etc.). |
| `whitebox_metrics_trend` | Get trend statistics for a metric. |
| `whitebox_metrics_rolling` | Get rolling-window statistics for a metric. |
| `whitebox_metrics_summary` | Get a summary of all metrics for a model. |
| `whitebox_metrics_delete` | Delete all metrics for a model. |

## Current limitations

- **Tool coverage.** The synchronous explanations endpoint and the bulk/compare/visualize/
  config-CRUD explanation endpoints aren't wrapped as tools yet — use the [SDK](/sdk/) or
  [REST API](/sdk/api-reference/#explainability-xai) for those. Within Alert Intelligence,
  four forecasting tools return placeholder data rather than a real prediction — see the note
  under [that section](#alert-intelligence).
- **Authentication.** In stdio mode, the email/password mode doesn't work with 2FA-enabled
  accounts, and a static JWT can't be refreshed — use an API key. In HTTP mode, sessions are
  always JWT-based and never refresh themselves; see [Connect over HTTP](#connect-over-http).

## Troubleshooting

**`whiteboxxai-mcp: no credentials configured` (exit code 2)**

No auth mode is configured. Set exactly one of `WHITEBOXXAI_MCP_API_KEY`,
`WHITEBOXXAI_MCP_JWT`, or the `WHITEBOXXAI_MCP_EMAIL` + `WHITEBOXXAI_MCP_PASSWORD` pair.

**Tool calls start failing with 401 after about 30 minutes**

You're on static-JWT mode and the token expired. Switch to `WHITEBOXXAI_MCP_API_KEY`, which
doesn't expire, or to the email/password mode, which refreshes itself.

**`TwoFactorNotSupportedError` on login**

The email/password account has 2FA enabled. Use an API key, or a service account with 2FA
disabled.

**A tool call hangs for ~30 seconds, then fails**

`WHITEBOXXAI_MCP_API_BASE_URL` isn't reachable from wherever the process is running. Confirm
the URL and check `WHITEBOXXAI_MCP_API_TIMEOUT`.

**A tool returns 403**

Your API key's scopes don't cover that operation. Check the key's scopes in **Profile → API
Keys** and issue a new one if needed.

## Security notes

- Prefer API-key authentication and revoke keys you're no longer using.
- No credential values are logged. Verbose mode logs request/response metadata only.
- The server never writes model weights or training data to disk — it only calls
  WhiteBoxXAI API endpoints.

## Related

- [API Keys](/account/api-keys/) — issuing and revoking the credential this server uses.
- [Python SDK](/sdk/) — the direct path for Python callers.
- [REST API Reference](/sdk/api-reference/) — the endpoints behind every tool above.
