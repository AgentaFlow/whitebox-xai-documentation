---
description: The WhiteBoxXAI MCP server (MCP Governance Layer) — install whiteboxxai-mcp or use the hosted endpoint, configure Claude Desktop, Claude Code, Cursor or VS Code, render MCP Apps cards, understand governance voting, and browse all 197 tools.
---

# MCP Server

The WhiteBoxXAI MCP server is the **MCP Governance Layer** for AI assistants and agents. It gives any
[Model Context Protocol](https://modelcontextprotocol.io/) client (Claude Desktop, Claude Code,
Cursor, VS Code, LangChain or Semantic Kernel agents, or your own harness) **197 tools** over your
WhiteBoxXAI data. With them, an assistant can register models, log predictions and LLM calls, check
drift, explain predictions, run bias audits, triage alerts, read ISO/IEC 42001 readiness and the risk
register, summarise governance review requests, and manage prompt versions.

Every tool calls the real WhiteBoxXAI API. None returns mocked or placeholder data. When something
can't be computed, the tool says so.

Writing Python application code? Use the [SDK](../sdk/index.md) instead. The MCP server is for
assistants and agents that *use* WhiteBoxXAI as a tool.

!!! tip "Let your coding agent set this up"
    The [`whitebox-mcp-server` skill](../agent-resources/skill-libraries.md) teaches an agent to
    install and configure the server and pick the right tool. Load it with `whitebox-governance-rules`.

## Two ways to connect

| | Local (stdio) | Hosted (Streamable HTTP) |
| --- | --- | --- |
| **What runs where** | `whiteboxxai-mcp` runs on your machine; your client starts it | Nothing to install; your client connects to `https://api.whiteboxxai.com/mcp/` |
| **Credential** | An API key in `WHITEBOXXAI_MCP_API_KEY` | An `Authorization: Bearer` header on every request: an API key or a personal sign-in token |
| **Governance voting** | Not available | Available with a personal sign-in token |
| **Best for** | Coding assistants and agents in your own environment | Clients that support remote MCP servers, and interactive voting |

## Install (local)

Requires Python 3.11 or later.

```bash
pip install whiteboxxai-mcp
```

Create an API key in **Settings → API keys** (administrators only; see [API Keys](../account/api-keys.md)),
then check the server starts:

```bash
export WHITEBOXXAI_MCP_API_KEY="wbx_live_..."
whiteboxxai-mcp
```

Your MCP client normally starts the command itself; you don't keep it running in a terminal.

### Configuration

| Variable | Default | Purpose |
| --- | --- | --- |
| `WHITEBOXXAI_MCP_API_KEY` | none (required) | Your API key (`wbx_live_...`) |
| `WHITEBOXXAI_MCP_API_BASE_URL` | `https://api.whiteboxxai.com` | Your API address. Business Cloud tenants use the address they were given. |
| `WHITEBOXXAI_MCP_API_TIMEOUT` | `30` | Request timeout, in seconds |
| `WHITEBOXXAI_MCP_LOG_LEVEL` | | Log level, for example `DEBUG` |

`whiteboxxai-mcp --verbose` (or `WHITEBOXXAI_MCP_VERBOSE=1`) logs request and response metadata, never
credential values.

The local server authenticates **only** with an API key. If none is set, it exits with code 2 and
`whiteboxxai-mcp: no credentials configured`.

## Connect a client

=== "Claude Desktop / Cursor / Windsurf"

    Add to the client's MCP config: `claude_desktop_config.json` for Claude Desktop,
    `~/.cursor/mcp.json` (or `.cursor/mcp.json` in a project) for Cursor.

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

=== "Claude Code"

    ```bash
    claude mcp add whiteboxxai \
      --env WHITEBOXXAI_MCP_API_KEY=wbx_live_... \
      -- whiteboxxai-mcp
    ```

    Or add the same `mcpServers` block as Claude Desktop to a project's `.mcp.json`. Don't commit a
    real key; reference an environment variable such as `${WHITEBOXXAI_MCP_API_KEY}` in the file.

=== "VS Code"

    VS Code uses a `servers` key in `.vscode/mcp.json`, and can prompt for the key instead of storing it:

    ```json
    {
      "inputs": [
        {
          "type": "promptString",
          "id": "whitebox-key",
          "description": "WhiteBoxXAI API key",
          "password": true
        }
      ],
      "servers": {
        "whiteboxxai": {
          "type": "stdio",
          "command": "whiteboxxai-mcp",
          "env": {
            "WHITEBOXXAI_MCP_API_KEY": "${input:whitebox-key}"
          }
        }
      }
    }
    ```

=== "Hosted (remote HTTP)"

    For clients that support remote MCP servers. With Claude Code:

    ```bash
    claude mcp add --transport http whiteboxxai https://api.whiteboxxai.com/mcp/ \
      --header "Authorization: Bearer wbx_live_..."
    ```

    In a JSON config that supports remote servers:

    ```json
    {
      "mcpServers": {
        "whiteboxxai": {
          "type": "http",
          "url": "https://api.whiteboxxai.com/mcp/",
          "headers": { "Authorization": "Bearer wbx_live_..." }
        }
      }
    }
    ```

=== "LangChain / Semantic Kernel / custom"

    Any MCP client library can start `whiteboxxai-mcp` over stdio with the environment above, or
    connect to the hosted endpoint with an `Authorization` header. For LangChain, the
    `langchain-mcp-adapters` package loads the tools into an agent.

More copy-paste definitions are on [Agent Config Definitions](../agent-resources/yaml-definitions.md).

### Hosted endpoint authentication

- Every request needs its own `Authorization: Bearer <token>` header: an API key, or a personal
  sign-in token. There's no fallback to environment variables, and a request without a valid header
  is refused with `401` before a session starts.
- A personal sign-in token expires after about 30 minutes and isn't refreshed by the server. Reconnect
  with a fresh token, or use an API key for long-running sessions.
- Activity sent with an API key is metered as [Governance Events](../get-started/governance-events.md)
  in the same way as SDK calls.

## MCP Apps: interactive cards

In clients that support MCP Apps, some results render as interactive cards inside the conversation
instead of raw JSON:

| Card | Shown for |
| --- | --- |
| **Explanation card** | `whitebox_explanations_generate`, `_get`, `_get_by_prediction` |
| **Alert triage card** | `whitebox_alerts_get_instance`, `whitebox_alerts_get_triage_context`, `whitebox_drift_get_report` |
| **Governance dossier card** | `whitebox_governance_list_review_requests`, `_get_review_request`, `_submit_decision` |
| **ISO/IEC 42001 status card** | `whitebox_compliance_iso42001_score`, `_gaps`, `_status` |
| **Model version diff card** | `whitebox_models_compare_versions`, `_version_changelog`, `_architecture_timeline` |

How the cards are built:

- Each card is a sandboxed `ui://whitebox/...` resource with a strict content security policy: it
  can't load anything from the network and holds no credentials. Buttons on a card ask your client
  to call a tool; they don't call WhiteBoxXAI directly.
- The card and the text result carry the same data and the same plain-language verdict. Clients
  without MCP Apps support get the full JSON result.
- Explanation cards replace common personal data patterns, such as email addresses and phone
  numbers, with `[REDACTED:<TYPE>]` before display.

## Governance voting

The governance tools let an assistant prepare a review: list what's waiting, open a request's
dossier (board, submitter, decisions, comments, vote tally and its **evidence snapshot hash**), and
summarise it. **Casting the vote stays with a person:**

- `whitebox_governance_submit_decision` is an **app-only** tool. It's hidden from the model and only
  offered as a button on the governance dossier card, so an assistant can't vote on its own.
- A vote needs **your own sign-in token** on the hosted endpoint. An organization API key is refused,
  and so is the local stdio server, which only uses API keys.
- Every vote needs a current **authenticator code**, and two-factor authentication must be on.
- The vote carries the evidence snapshot hash of the dossier you saw. If the request changed since,
  it's refused with `409`, and you review the current version first.

See [Reviews & Decisions](../user-guide/governance.md#vote-with-an-authenticator-code).

## Example prompts

| Ask the assistant | It calls |
| --- | --- |
| "What models do we have?" | `whitebox_models_list` |
| "Has credit_scoring drifted this month? Keep a record." | `whitebox_drift_detect`, then `whitebox_drift_create_report` |
| "Why was this prediction declined?" | `whitebox_explanations_get_by_prediction` or `whitebox_explanations_generate` |
| "What's firing right now, and why?" | `whitebox_alerts_list_instances`, `whitebox_alerts_get_triage_context` |
| "Where are we on ISO 42001?" | `whitebox_compliance_iso42001_status` |
| "What's waiting for my review?" | `whitebox_governance_list_review_requests` |
| "How did v3 differ from v2?" | `whitebox_models_compare_versions` |
| "Roll back the support-bot prompt to the last verified version." | `whitebox_prompts_rollback` |

## Tool reference

Tools are named `whitebox_<domain>_<action>`, across 18 domains. Required arguments are listed; see
each tool's input schema in your client for optional ones.

### Models

15 tools.

| Tool | What it does | Required arguments |
| --- | --- | --- |
| `whitebox_models_architecture_timeline` | Get the architecture changes timeline across all versions of a model. | `model_id` |
| `whitebox_models_archive` | Archive a model (soft delete; can be restored later). | `model_id` |
| `whitebox_models_compare_versions` | Compare two Git-backed model versions, returning architecture changes, parameter deltas, and breaking change flags. | `version1_id`, `version2_id` |
| `whitebox_models_delete` | Permanently delete a model. | `model_id` |
| `whitebox_models_get` | Get a registered model's full record by its model_id. | `model_id` |
| `whitebox_models_latest` | Get the most recently created version of a model by name. | `model_name` |
| `whitebox_models_list` | List registered models, optionally filtered by status/type/tags/search text. | — |
| `whitebox_models_list_git_versions` | List Git-backed version sources for a model, newest first. | `model_id` |
| `whitebox_models_register` | Register a new model in WhiteBoxXAI's model registry. | `name`, `model_type`, `features`, `baseline_metrics` |
| `whitebox_models_restore` | Restore a previously archived model to active status. | `model_id` |
| `whitebox_models_update` | Partially update a model's metadata. | `model_id` |
| `whitebox_models_update_baseline` | Refresh a model's baseline metrics and profile, for example after retraining. | `model_id`, `baseline_metrics` |
| `whitebox_models_update_status` | Update a model's status: ACTIVE, INACTIVE, DEPRECATED, or ARCHIVED. | `model_id`, `new_status` |
| `whitebox_models_version_changelog` | Generate a markdown changelog between two Git-backed model versions. | `version1_id`, `version2_id` |
| `whitebox_models_versions` | List all versions of a model by name, newest first. | `model_name` |

### Predictions

6 tools.

| Tool | What it does | Required arguments |
| --- | --- | --- |
| `whitebox_predictions_get` | Get a single logged prediction by its prediction_id. | `prediction_id` |
| `whitebox_predictions_log` | Log a single model prediction for monitoring. | `model_id`, `input_data`, `output_data` |
| `whitebox_predictions_log_batch` | Log up to 1000 predictions in a single batch call. | `model_id`, `predictions` |
| `whitebox_predictions_query` | Query logged predictions for a model, optionally within a time range. | `model_id` |
| `whitebox_predictions_recent` | Get the most recent predictions for a model (max 100). | `model_id` |
| `whitebox_predictions_stats` | Get prediction volume/latency statistics for a model. | `model_id` |

### Prompts

13 tools.

| Tool | What it does | Required arguments |
| --- | --- | --- |
| `whitebox_prompts_advance_stage` | Advance a prompt version through the lifecycle stages (design -> verify -> deploy -> retired). | `template_id`, `version_id`, `target_stage` |
| `whitebox_prompts_compute_drift` | Compute prompt drift analysis comparing production completions with baseline expectations. | `template_id`, `version_id` |
| `whitebox_prompts_create_template` | Create a new prompt template container for versioning. | `name` |
| `whitebox_prompts_create_version` | Create a new governed prompt version in 'design' lifecycle stage. | `template_id`, `user_message_template` |
| `whitebox_prompts_deploy` | Promote a prompt version to active production deployment. | `template_id`, `version_id` |
| `whitebox_prompts_get_drift_report` | Get a specific prompt drift report with metrics, divergence, and auditor verdict. | `template_id`, `version_id`, `report_id` |
| `whitebox_prompts_get_incidents` | Get security incidents and alerts associated with a prompt version. | `template_id`, `version_id` |
| `whitebox_prompts_get_template` | Get details of a specific prompt template by ID. | `template_id` |
| `whitebox_prompts_get_version` | Get details, lifecycle stage, and verification status of a prompt version. | `template_id`, `version_id` |
| `whitebox_prompts_list_drift_reports` | List historical prompt drift reports for a prompt version. | `template_id`, `version_id` |
| `whitebox_prompts_list_templates` | List all prompt governance templates for the authenticated tenant. | — |
| `whitebox_prompts_list_versions` | List all append-only governed versions for a prompt template. | `template_id` |
| `whitebox_prompts_rollback` | Emergency rollback to a target version or the most recently verified version. | `template_id` |

### Drift

5 tools.

| Tool | What it does | Required arguments |
| --- | --- | --- |
| `whitebox_drift_create_report` | Run drift analysis and persist the result as a drift report. | `model_id` |
| `whitebox_drift_detect` | Run an ad-hoc data drift check for a model (not persisted as a report). | `model_id` |
| `whitebox_drift_get_report` | Get a specific drift report, including per-feature statistics. | `model_id`, `report_id` |
| `whitebox_drift_list_reports` | List drift reports for a model, most recent first. | `model_id` |
| `whitebox_drift_trend` | Get a model's drift trend over time (1-90 days). | `model_id` |

### Fairness

6 tools.

| Tool | What it does | Required arguments |
| --- | --- | --- |
| `whitebox_fairness_audit` | Run a bias/fairness audit on model predictions across demographic groups. | `model_id`, `sensitive_attributes`, `y_true`, `y_pred`, `group_data` |
| `whitebox_fairness_bias_history` | Get a model's historical bias/fairness trend (1-365 days). | `model_id` |
| `whitebox_fairness_get_audit` | Get full bias audit results by audit_id. | `audit_id` |
| `whitebox_fairness_latest_audit` | Get the most recent bias audit for a model. | `model_id` |
| `whitebox_fairness_list_audits` | List bias audit summaries, optionally filtered to one model. | — |
| `whitebox_fairness_metric_history` | Get history for one specific fairness metric on a model. | `model_id`, `metric_type` |

### Explanations

13 tools.

| Tool | What it does | Required arguments |
| --- | --- | --- |
| `whitebox_explanations_compare` | Compare 2-20 explanations against each other. | `explanation_ids` |
| `whitebox_explanations_delete_config` | Delete a model's explanation configuration. | `model_id` |
| `whitebox_explanations_generate` | Generate a SHAP/LIME explanation and block until it's computed. | `model_id`, `instance` |
| `whitebox_explanations_generate_async` | Start real SHAP/LIME explanation generation for a model instance. | `model_id`, `instance` |
| `whitebox_explanations_generate_bulk` | Start explanation generation for multiple instances at once (non-blocking). | `model_id`, `instances` |
| `whitebox_explanations_get` | Get an explanation by ID, including its status and (once ready) results. | `explanation_id` |
| `whitebox_explanations_get_by_prediction` | Get the explanation associated with a prediction, if one exists. | `prediction_id` |
| `whitebox_explanations_get_config` | Get a model's explanation configuration. | `model_id` |
| `whitebox_explanations_list_by_model` | List explanations for a model, optionally filtered by method or status. | `model_id` |
| `whitebox_explanations_set_config` | Create or update a model's explanation configuration (upsert). | `model_id` |
| `whitebox_explanations_stats` | Get aggregated explanation statistics for a model (counts by method/status, average computation time). | `model_id` |
| `whitebox_explanations_visualize` | Get visualization data for a single explanation. | `explanation_id`, `plot_type` |
| `whitebox_explanations_visualize_multi` | Get visualization data spanning multiple explanations. | `explanation_ids`, `plot_type` |

### Risk register

3 tools.

| Tool | What it does | Required arguments |
| --- | --- | --- |
| `whitebox_risk_register_get` | Get a single AI risk register entry by ID. | `entry_id` |
| `whitebox_risk_register_list` | List AI risk register entries, optionally filtered. | — |
| `whitebox_risk_register_portfolio` | Get the likelihood x impact heat map and top open risks across the organization's risk register. | — |

### Governance

5 tools.

| Tool | What it does | Required arguments |
| --- | --- | --- |
| `whitebox_governance_get_review_request` | Get a specific governance review request dossier (board, submitter, decisions, comments, vote tally, and SHA-256 evidence snapshot hash). | `request_id` |
| `whitebox_governance_list_boards` | List governance review boards for the organization. | — |
| `whitebox_governance_list_review_requests` | List governance review requests, optionally filtered. | — |
| `whitebox_governance_raci_grid` | Get the RACI grid: review requests and their Responsible / Accountable / Consulted / Informed assignments (ISO 42001 Clause 5.3, Annex A.3), optionally scoped to one board. | — |
| `whitebox_governance_submit_decision` | Record a board member's vote on a governance review request. | `request_id`, `decision`, `evidence_snapshot_hash`, `totp_code` |

### Compliance

5 tools.

| Tool | What it does | Required arguments |
| --- | --- | --- |
| `whitebox_compliance_get_framework_controls` | Get the full control catalog for one compliance framework. | `framework` |
| `whitebox_compliance_iso42001_gaps` | Get the organization's open ISO/IEC 42001 compliance gaps. | — |
| `whitebox_compliance_iso42001_score` | Get the organization's ISO/IEC 42001 readiness score and clause breakdown. | — |
| `whitebox_compliance_iso42001_status` | Get a composite ISO/IEC 42001 readiness scorecard, open gap matrix, and linked AI risk register entries in a single call for the `ui://whitebox/iso42001-status-card` widget. | — |
| `whitebox_compliance_list_frameworks` | List all supported compliance frameworks in the Control Catalog (ISO/IEC 42001, GDPR, CCPA, EU AI Act, NIST AI RMF) with control counts. | — |

### LLM explainability

13 tools.

| Tool | What it does | Required arguments |
| --- | --- | --- |
| `whitebox_llm_xai_attention` | Analyze transformer attention patterns for a prompt/completion pair. | `prompt` |
| `whitebox_llm_xai_batch_analyze` | Run multiple explanation analyses on a single LLM log in one call (more efficient than calling each analysis tool separately). | `llm_log_id` |
| `whitebox_llm_xai_counterfactuals` | Generate minimal prompt edits that change the model's output, useful for understanding decision boundaries. | `prompt`, `original_output` |
| `whitebox_llm_xai_debug_prompt` | Assess prompt quality across multiple dimensions and get improvement recommendations. | `prompt` |
| `whitebox_llm_xai_get_explanation` | Get a specific attention/token-importance/sensitivity/counterfactual explanation by ID. | `explanation_id` |
| `whitebox_llm_xai_get_prompt_analysis` | Get a specific prompt debug analysis by ID. | `analysis_id` |
| `whitebox_llm_xai_list_explanations` | List explanations, optionally filtered by LLM log, type (attention_analysis/token_importance/sensitivity_analysis/ counterfactual_generation), or model. | — |
| `whitebox_llm_xai_list_prompt_analyses` | List prompt analyses, optionally filtered by similar prompt text, model, or minimum quality score (0-1). | — |
| `whitebox_llm_xai_prompt_sensitivity` | Test prompt robustness by perturbing it and measuring output stability. | `prompt`, `baseline_output` |
| `whitebox_llm_xai_stats` | Get aggregated LLM-XAI usage statistics (counts by explanation type, average computation time, prompt quality). | — |
| `whitebox_llm_xai_token_importance` | Calculate token-level importance scores for a prompt/completion pair. | `prompt`, `completion` |
| `whitebox_llm_xai_visualize_attention` | Get attention-heatmap visualization data for an explanation (for rendering with Plotly/D3.js-style tools). | `explanation_id` |
| `whitebox_llm_xai_visualize_token_importance` | Get token-importance bar-chart visualization data for an explanation. | `explanation_id` |

### LLM monitoring

16 tools.

| Tool | What it does | Required arguments |
| --- | --- | --- |
| `whitebox_llm_monitoring_cleanup_logs` | Delete LLM logs older than `days`. | — |
| `whitebox_llm_monitoring_cost_breakdown` | Get detailed cost breakdown (platform-wide aggregate). | — |
| `whitebox_llm_monitoring_cost_threshold_alert` | Check whether costs over the recent period exceed a threshold. | — |
| `whitebox_llm_monitoring_error_rate_alert` | Check whether the error rate over the recent period exceeds a threshold. | — |
| `whitebox_llm_monitoring_get_log` | Get a single LLM log by ID. | `log_id` |
| `whitebox_llm_monitoring_latency_threshold_alert` | Check whether average latency over the recent period exceeds a threshold. | — |
| `whitebox_llm_monitoring_log_call` | Log a single LLM call with real token/cost/latency data. | `provider`, `model_name`, `prompt`, `latency_ms` |
| `whitebox_llm_monitoring_log_calls_batch` | Log multiple LLM calls in one request (max 1000). | `logs` |
| `whitebox_llm_monitoring_performance` | Get LLM latency/throughput performance metrics. | — |
| `whitebox_llm_monitoring_query_logs` | Query LLM logs with filters and pagination. | — |
| `whitebox_llm_monitoring_recent` | Get recent LLM logs (requires a model_id owned by the caller's org). | — |
| `whitebox_llm_monitoring_session_logs` | Get all LLM logs for a specific session. | `session_id` |
| `whitebox_llm_monitoring_trends_costs` | Get cost trend over time (platform-wide; granularity: hour/day/week). | — |
| `whitebox_llm_monitoring_trends_tokens` | Get token usage trend over time (granularity: hour/day/week). | — |
| `whitebox_llm_monitoring_usage_stats` | Get aggregated LLM usage statistics (tokens, cost, latency, by_model). | — |
| `whitebox_llm_monitoring_usage_stats_detailed` | Get detailed, grouped usage statistics (granularity: hour/day/week/month). | `start_time`, `end_time` |

### RAG

10 tools.

| Tool | What it does | Required arguments |
| --- | --- | --- |
| `whitebox_rag_create_evaluation` | Create a RAG evaluation, aggregating already-logged retrievals for each query. | `name`, `queries` |
| `whitebox_rag_get_evaluation` | Get a specific RAG evaluation by ID. | `evaluation_id` |
| `whitebox_rag_get_retrieval` | Get a specific retrieval log by ID. | `log_id` |
| `whitebox_rag_list_evaluations` | List RAG evaluations with optional filters. | — |
| `whitebox_rag_list_retrievals` | Query retrieval logs for a model. | `model_id` |
| `whitebox_rag_log_retrieval` | Log a retrieval operation, with real quality metrics computed server-side when ground_truth_ids is supplied. | `query`, `results`, `top_k`, `retrieval_method` |
| `whitebox_rag_metrics_precision` | Get Precision@K metrics for the given K values (default [1, 3, 5, 10]). | — |
| `whitebox_rag_metrics_relevance` | Get average relevance metrics (context/answer/faithfulness). | — |
| `whitebox_rag_stats` | Get aggregated RAG statistics (precision/recall/MRR/NDCG/relevance/latency). | — |
| `whitebox_rag_trends` | Get RAG quality metrics trend over time (granularity: hour/day/week). | — |

### Safety

11 tools.

| Tool | What it does | Required arguments |
| --- | --- | --- |
| `whitebox_safety_analyze` | Analyze content for toxicity, PII, and harmful content. | `content` |
| `whitebox_safety_analyze_batch` | Analyze multiple content items in one request (max 100). | `contents` |
| `whitebox_safety_create_threshold` | Create a safety threshold configuration controlling when content is flagged as unsafe. | `name` |
| `whitebox_safety_delete_threshold` | Delete a safety threshold configuration. | `threshold_id` |
| `whitebox_safety_get_score` | Get a specific safety score by ID. | `score_id` |
| `whitebox_safety_get_scores` | Query safety analysis scores with filters. | — |
| `whitebox_safety_get_threshold` | Get a specific safety threshold by ID. | `threshold_id` |
| `whitebox_safety_list_thresholds` | List safety thresholds for the caller's organization. | — |
| `whitebox_safety_stats` | Get aggregated safety statistics (per-category toxicity averages, flagged categories, PII types). | — |
| `whitebox_safety_trends` | Get content-safety trend over time (granularity: hour/day/week). | — |
| `whitebox_safety_update_threshold` | Update a safety threshold configuration. | `threshold_id` |

### Alerts

13 tools.

| Tool | What it does | Required arguments |
| --- | --- | --- |
| `whitebox_alerts_acknowledge` | Acknowledge an active alert instance. | `alert_id`, `user_id` |
| `whitebox_alerts_create_rule` | Create an alert rule with threshold or anomaly-based conditions. | `name`, `alert_type`, `severity`, `conditions` |
| `whitebox_alerts_delete_rule` | Delete an alert rule. | `rule_id` |
| `whitebox_alerts_evaluate_rule` | Test whether an alert rule would trigger given metric values, without creating a real alert instance. | `rule_id` |
| `whitebox_alerts_get_instance` | Get a specific triggered alert instance by ID. | `alert_id` |
| `whitebox_alerts_get_rule` | Get a specific alert rule by ID. | `rule_id` |
| `whitebox_alerts_get_triage_context` | Get a triggered alert instance together with its model's latest drift reports and a plain-language triage verdict. | `alert_id` |
| `whitebox_alerts_list_instances` | List triggered alert instances with optional filters. | — |
| `whitebox_alerts_list_rules` | List alert rules for the caller's organization. | — |
| `whitebox_alerts_resolve` | Resolve an alert instance. | `alert_id`, `user_id` |
| `whitebox_alerts_snooze` | Snooze an active alert instance for `snooze_minutes` (max 1 week). | `alert_id`, `snooze_minutes` |
| `whitebox_alerts_statistics` | Get alert statistics for the organization (counts, severity distribution, mean time to acknowledge/resolve) over the last `hours`. | — |
| `whitebox_alerts_update_rule` | Update an alert rule. | `rule_id` |

### Alert intelligence

20 tools.

| Tool | What it does | Required arguments |
| --- | --- | --- |
| `whitebox_alert_intelligence_adjust_priority` | Manually override a calculated priority. | `alert_id`, `new_priority_score`, `new_priority_level`, `adjustment_reason` |
| `whitebox_alert_intelligence_analyze_root_cause` | Analyze potential root causes (data drift, model degradation, feature anomalies, system issues) for an alert. | `alert_id` |
| `whitebox_alert_intelligence_apply_fatigue_reduction` | Apply automated throttling/grouping/snoozing to reduce alert noise. | — |
| `whitebox_alert_intelligence_calculate_priority` | Compute a weighted priority score (0-100) and level (critical/high/medium/low) for an alert. | `alert_id` |
| `whitebox_alert_intelligence_deactivate_pattern` | Mark a pattern inactive (resolved or no longer relevant). | `pattern_id` |
| `whitebox_alert_intelligence_detect_correlations` | Detect correlations for an alert (temporal, model, feature, and pattern-based relationships to other alerts). | `alert_id` |
| `whitebox_alert_intelligence_detect_patterns` | Detect recurring temporal/frequency/correlation/anomaly patterns in alerts. | — |
| `whitebox_alert_intelligence_detect_seasonality` | Detect hour-of-day seasonality in alert volume. | — |
| `whitebox_alert_intelligence_enhanced_trend_analysis` | Combine alert-volume time series, an hour-of-day seasonality estimate, a linear trend prediction with an R²-based confidence score, and z-score anomaly detection, all computed from the organization's actual alert history. | — |
| `whitebox_alert_intelligence_forecast_volume` | Forecast future alert volume with confidence intervals. | — |
| `whitebox_alert_intelligence_get_correlation_clusters` | Get clusters of highly correlated alerts: useful for spotting widespread issues or cascading failures. | — |
| `whitebox_alert_intelligence_get_pattern` | Get a specific alert pattern's details. | `pattern_id` |
| `whitebox_alert_intelligence_get_priority_distribution` | Get a breakdown of alerts by priority level. | — |
| `whitebox_alert_intelligence_get_priority_queue` | Get the prioritized alert queue for triage and assignment. | — |
| `whitebox_alert_intelligence_get_similar_incidents` | Find past alerts sharing the same root-cause type: useful for learning from historical resolutions. | `cause_type` |
| `whitebox_alert_intelligence_get_throttle_recommendations` | Get recommended alert-throttling rules to reduce fatigue without missing critical issues. | — |
| `whitebox_alert_intelligence_ml_grouping` | Cluster alerts via ML (kmeans/dbscan/hierarchical). | — |
| `whitebox_alert_intelligence_predict_trends` | Forecast alert volume/severity/type for `time_horizon` (next_hour/next_day/next_week). | — |
| `whitebox_alert_intelligence_temporal_grouping` | Group alerts occurring in the same time window with similar characteristics. | — |
| `whitebox_alert_intelligence_verify_root_cause` | Mark a previously computed root cause as verified or rejected. | `root_cause_id`, `verified` |

### Alert management

13 tools.

| Tool | What it does | Required arguments |
| --- | --- | --- |
| `whitebox_alert_management_acknowledge` | Acknowledge an alert (hits /alert-management/alerts/{id}/acknowledge; distinct from whitebox_alerts_acknowledge). | `alert_id` |
| `whitebox_alert_management_create_comment` | Add a comment to an alert (comment_type: note/action/resolution). | `alert_id`, `comment` |
| `whitebox_alert_management_detect_fatigue` | Detect alert-fatigue indicators and get recommendations for reducing alert noise. | — |
| `whitebox_alert_management_get_analytics` | Get comprehensive alert analytics (counts, MTTA/MTTR, distributions, escalation metrics) for the caller's organization. | — |
| `whitebox_alert_management_get_comments` | List all comments on an alert. | `alert_id` |
| `whitebox_alert_management_get_grouping` | Group active alerts by a dimension: "type", "severity", "model", or "rule": useful for spotting hotspots. | `group_by` |
| `whitebox_alert_management_get_history` | Get the full action timeline (triggered/acknowledged/snoozed/ resolved/escalated/commented) for an alert. | `alert_id` |
| `whitebox_alert_management_get_trend` | Get alert volume as a time series between `start` and `end` (ISO 8601 timestamps), bucketed by `interval` (hour/day/week). | `start`, `end` |
| `whitebox_alert_management_get_velocity` | Get alert generation rate trends (hour-over-hour, day-over-day, rolling averages) for the caller's organization. | — |
| `whitebox_alert_management_process_pending_escalations` | Manually trigger processing of pending escalations (normally a scheduled background task). | — |
| `whitebox_alert_management_resolve` | Resolve an alert (hits /alert-management/alerts/{id}/resolve; distinct from whitebox_alerts_resolve). | `alert_id` |
| `whitebox_alert_management_snooze` | Snooze an alert for `snooze_minutes` (max 1 week; hits /alert-management/alerts/{id}/snooze, distinct from whitebox_alerts_snooze). | `alert_id`, `snooze_minutes` |
| `whitebox_alert_management_unsnooze_expired_alerts` | Manually trigger reactivation of alerts whose snooze period has expired (normally a scheduled background task). | — |

### Agent workflows

18 tools.

| Tool | What it does | Required arguments |
| --- | --- | --- |
| `whitebox_agent_workflows_analytics` | Get workflow analytics (tokens, costs, execution counts, durations). | `workflow_id` |
| `whitebox_agent_workflows_bottlenecks` | Identify workflow bottlenecks: slowest agents, slowest tasks, failing agents. | `workflow_id` |
| `whitebox_agent_workflows_complete` | Mark a workflow completed/failed/cancelled and optionally trigger async analytics (metrics, cost, bottlenecks, timeline). | `workflow_id`, `status` |
| `whitebox_agent_workflows_cost_breakdown` | Get per-agent cost breakdown (tokens, cost, execution count, avg duration). | `workflow_id` |
| `whitebox_agent_workflows_create_and_start` | Create a new multi-agent workflow. | `name`, `framework` |
| `whitebox_agent_workflows_create_execution` | Log that an agent started executing. | `workflow_id`, `agent_id` |
| `whitebox_agent_workflows_create_task` | Create a workflow task. | `workflow_id`, `task_name` |
| `whitebox_agent_workflows_delete` | Delete a workflow and all related agents/executions/tasks/interactions. | `workflow_id` |
| `whitebox_agent_workflows_get` | Get a workflow by ID. | `workflow_id` |
| `whitebox_agent_workflows_list` | List workflows for the organization. | — |
| `whitebox_agent_workflows_list_agents` | List all agents registered in a workflow. | `workflow_id` |
| `whitebox_agent_workflows_list_interactions` | List all agent-to-agent interactions in a workflow, chronologically. | `workflow_id` |
| `whitebox_agent_workflows_list_tasks` | List all tasks in a workflow, with status, assignments, and results. | `workflow_id` |
| `whitebox_agent_workflows_log_interaction` | Log agent-to-agent communication. | `workflow_id`, `interaction_type` |
| `whitebox_agent_workflows_register_agent` | Register an agent in a workflow. | `workflow_id`, `name` |
| `whitebox_agent_workflows_start` | Mark a workflow as running. | `workflow_id` |
| `whitebox_agent_workflows_timeline` | Get a chronological timeline of workflow events (start/end, agent executions, task updates, interactions). | `workflow_id` |
| `whitebox_agent_workflows_update_task_status` | Update a task's status. | `task_id`, `status` |

### Metrics

12 tools.

| Tool | What it does | Required arguments |
| --- | --- | --- |
| `whitebox_metrics_aggregate` | Get metrics aggregated over a `period`, such as daily or weekly. | `model_id`, `period` |
| `whitebox_metrics_calculate_classification` | Calculate classification metrics (accuracy/precision/recall/F1/etc.) from predictions, without persisting anything. | `y_true`, `y_pred` |
| `whitebox_metrics_calculate_regression` | Calculate regression metrics (MAE/MSE/RMSE/R2/etc.) from predictions, without persisting anything. | `y_true`, `y_pred` |
| `whitebox_metrics_create` | Create a new metric record for a model. | `model_id`, `metric_type`, `metric_name`, `metric_value` |
| `whitebox_metrics_create_batch` | Create multiple metrics at once (max 100). | `model_id`, `metrics` |
| `whitebox_metrics_delete` | Delete all metrics for a model. | `model_id` |
| `whitebox_metrics_get_model_metrics` | Get metrics recorded for a model, with optional type/date filters. | `model_id` |
| `whitebox_metrics_latest` | Get the most recent metric of a specific type for a model. | `model_id`, `metric_type` |
| `whitebox_metrics_rolling` | Get rolling-window statistics for a metric over `window_days`. | `model_id`, `metric_type` |
| `whitebox_metrics_summary` | Get a summary of all metrics for a model over the last `days`. | `model_id` |
| `whitebox_metrics_timeseries` | Get time series data for a metric over a required date range. | `model_id`, `metric_type`, `start_date`, `end_date` |
| `whitebox_metrics_trend` | Get trend statistics for a metric over `lookback_days`. | `model_id`, `metric_type` |

## Current limitations

- **No report generation tools.** Generate reports with the [SDK](../sdk/index.md) or the
  dashboard. Over MCP, an assistant can create and read drift reports and read the evidence reports
  draw on.
- **Not every administrative endpoint is a tool.** Escalation-policy management and bulk alert
  actions are left out on purpose. If a tool doesn't exist, the assistant should say so rather than
  simulate it.
- **Alert rules aren't evaluated automatically yet.** `whitebox_alerts_evaluate_rule` tests a rule
  against values you supply; see [Needs Attention](../user-guide/needs-attention.md#alert-rules).
- **Workflow analytics can be slow.** `whitebox_agent_workflows_cost_breakdown`, `_bottlenecks` and
  `_timeline` can take up to about 30 seconds.
- **Hosted sessions with a personal sign-in token expire** after about 30 minutes; reconnect with a
  fresh token, or use an API key where you don't need to vote.

## Troubleshooting

**`whiteboxxai-mcp: no credentials configured` (exit code 2).** Set `WHITEBOXXAI_MCP_API_KEY` in the
environment the client starts the server with, not only in your shell.

**`401 Unauthorized`.** The key is mistyped, revoked or expired, or (on the hosted endpoint) the
`Authorization` header is missing or the sign-in token has expired.

**`403` on a vote.** Votes need your own sign-in token, two-factor authentication and an authenticator
code. An API key can't vote.

**`404` on an id.** The record isn't in your organization. Check the id rather than retrying with
other credentials.

**`429`.** Either the rate limit (wait for `Retry-After`) or, on the Free plan, the month's
[Governance Events](../get-started/governance-events.md#when-you-reach-the-limit) are used up.

**Calls hang, then time out.** `WHITEBOXXAI_MCP_API_BASE_URL` isn't reachable from where the server
runs. Check the address and `WHITEBOXXAI_MCP_API_TIMEOUT`.

## Security notes

- Use one API key per client or agent, so you can revoke one without breaking the others.
- Credential values are never logged; verbose mode logs request metadata only.
- The server doesn't write model weights or training data anywhere. It only calls the WhiteBoxXAI API.
- An API key acts with the permissions of the administrator who created it. See
  [API Keys](../account/api-keys.md#what-a-key-can-do).

## Related

- [Agent Skills](../agent-resources/skill-libraries.md) and [AI Prompts](../agent-resources/prompts.md)
- [API Keys](../account/api-keys.md)
- [Python SDK](../sdk/index.md) and the [REST API reference](../sdk/api-reference.md)
