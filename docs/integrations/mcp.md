# MCP Server

[MCP (Model Context Protocol)](https://modelcontextprotocol.io/) is an open protocol for
connecting AI clients — Claude Desktop, Claude Code, LangChain agents, custom agent
harnesses — to external tools and data.

WhiteBoxXAI ships an MCP server so any MCP-compatible client can register models, log
predictions, run drift detection, and run bias/fairness audits without writing integration
code. It's the recommended path for non-Python clients and for agents that need to reach
WhiteBoxXAI as a tool. If you're writing Python, use the [SDK](/sdk/) directly
instead.

Every tool calls the real WhiteBoxXAI API through the SDK. None of them return mocked or
placeholder data.

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

The server speaks **stdio** only — your MCP client spawns the `whiteboxxai-mcp` process and
talks to its stdin/stdout. There is no hosted HTTP endpoint yet; see
[Current limitations](#current-limitations).

## Authentication

Configure exactly one of these modes:

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

## Tool reference

26 tools across four domains, all prefixed `whitebox_`.

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

## Current limitations

- **Tool coverage.** Only models, predictions, drift, and bias/fairness tools ship today.
  Explanations (SHAP/LIME), LLM/RAG observability, safety, alerts, and multi-agent workflow
  tools are follow-on milestones. To generate explanations now, use the
  [SDK](/sdk/) or the [REST API](/sdk/api-reference/#explainability-xai).
- **Transport.** stdio only. A hosted HTTP endpoint — so clients wouldn't need to run their
  own process — needs a per-connection authentication design, since today's model assumes
  one fixed credential per server process set via environment variables.
- **Authentication.** The email/password mode doesn't work with 2FA-enabled accounts, and a
  static JWT can't be refreshed. Use an API key.

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
