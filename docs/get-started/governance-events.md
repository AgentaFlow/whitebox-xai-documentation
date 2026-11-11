---
description: Exactly what counts as a WhiteBoxXAI Governance Event and a Governed AI System, how batches, agent workflows and run grouping are counted, what is free, and what the 429 and 403 limit responses mean.
---

# Governance Events & Governed AI Systems

WhiteBoxXAI plans are sized in two units: how many **AI systems** you govern, and how much **AI
activity** is recorded. Neither is "API calls". Reading data, configuring things, generating reports
and using the dashboard cost nothing. Only recording new AI activity is counted, and the rules are
set per endpoint so a batch or an agent run doesn't cost more than the activity it records.

For plan allowances, see [Plans & Limits](plans.md).

## What is one Governance Event

A Governance Event is one unit of AI activity that WhiteBoxXAI records:

| Activity | SDK call | Events |
| --- | --- | --- |
| Logged prediction | `client.predictions.log` | 1 |
| LLM interaction | `client.llm.log_call` | 1 |
| LLM call recorded through a [LiteLLM gateway](../integrations/litellm.md) | none: the gateway sends it | 1 per call stored |
| RAG retrieval | `client.rag.log_retrieval` | 1 |
| Safety scan | `client.safety.analyze` | 1 |
| Explanation (SHAP, LIME, DiCE) | `client.explanations.generate` / `generate_async` | 1 |
| LLM explanation (attention, token importance, sensitivity, counterfactuals, prompt debugging) | `client.llm_xai.*` | 1 |
| Several analyses of one LLM log in a single `batch-analyze` request | `client.llm_xai.batch_analyze` | 1 |
| Bias (fairness) audit | `client.fairness.audit` | 1 |
| Drift check or drift report | `client.drift.detect` / `create_report` | 1 |
| RAG evaluation | `client.rag.create_evaluation` | 1 |
| Prompt evaluation run | `POST /api/v1/evaluations/prompt` | 1 |
| Multi-agent workflow | `client.agent_workflows.create_and_start` | 1 per workflow |

### Batches: one event per record

Batch endpoints cost one event per record in the batch: `predictions.log_batch`,
`llm.log_calls_batch`, `safety.analyze_batch` and `explanations.generate_bulk`. Batching is neither
a discount nor a penalty. It costs exactly what sending the records one at a time would.

### LiteLLM gateway calls

A call recorded through a [LiteLLM gateway](../integrations/litellm.md) is one event, the same as a call you
log with the SDK. What is counted is what was **stored**, not what LiteLLM sent: a batch that LiteLLM
re-sends is recognised and costs nothing the second time, and a record WhiteBoxXAI cannot read is rejected
and costs nothing. A gateway's calls are not grouped by `X-WhiteBox-Run-Id`; each is its own event.

On the Free plan, a batch larger than what is left of the month is refused whole with `429`. LiteLLM does not
resend a refused batch, so those calls are lost; the gateway pane counts them and says so.

### Agent workflows: one event per workflow

A multi-agent workflow costs **one event when it's created**. Registering its agents, recording
tasks, executions and agent-to-agent interactions, and completing it are free, however many there
are.

### Grouped runs: one event per 100 records

Predictions, LLM interactions, RAG retrievals and safety scans that belong to the same end-to-end AI
run can share events. Send them with the same `X-WhiteBox-Run-Id` request header, and every **100
records** with that run id cost **one event**. A 12-call agent run is one event; a 250-record run
is three.

The run id is an opaque value you choose: up to 128 characters of letters, digits, `.`, `_`, `:` or
`-`. A header that doesn't match that shape is ignored, and those records are counted individually.
Explanations and evaluations are not groupable; each one is its own event.

In the Python SDK, wrap the run in `governance_run()` and the header is added for you:

```python
import os

from whiteboxxai import WhiteBoxXAI, governance_run

client = WhiteBoxXAI(api_key=os.environ["WHITEBOX_API_KEY"])

with governance_run() as run_id:          # or governance_run("ticket-4821") to choose the id
    client.llm.log_call(
        provider="anthropic",
        model_name="claude-sonnet-4-5",
        prompt=prompt,
        completion=answer,
        latency_ms=812.0,
        prompt_tokens=412,
        completion_tokens=96,
    )
    client.safety.analyze(content=answer, content_type="completion")
```

Every SDK request made inside the block carries the same run id. Pass your own id to group records
sent from several processes.

You don't need `governance_run()` when:

- you use the **LangChain callback handler**: it groups each root chain run automatically
- you log a **multi-agent workflow** (CrewAI, LangChain agents and others): the whole workflow is
  already one event

Over plain HTTP, set the header yourself:

```bash
curl -X POST https://api.whiteboxxai.com/api/v1/llm/logs \
  -H "Authorization: Bearer $WHITEBOX_API_KEY" \
  -H "X-WhiteBox-Run-Id: ticket-4821" \
  -H "Content-Type: application/json" \
  -d '{"provider": "openai", "model_name": "gpt-4o", "prompt": "...", "latency_ms": 640}'
```

## What is free

None of these count as Governance Events:

- **Everything you do in the dashboard.** Only requests authenticated with an API key (SDK, REST
  API, MCP server) are metered.
- Registering, updating, archiving and restoring models
- Configuration: drift settings, explanation settings, safety thresholds, alert rules, notification
  channels, trust score weights, brand profiles
- Alerts, review boards, review requests, the risk register, and compliance evidence
- Generating, scheduling, sharing and downloading reports
- Every `GET` request, and query-style `POST` requests that read existing records (for example
  `predictions/query`, `llm/logs/query`, `explanations/compare`, `evaluations/prompt/compare`)
- Performance metrics and metric calculators (`client.metrics.*`)
- Authoring golden datasets, prompt templates and prompt versions
- Requests the API rejects. Events are only counted for successful responses.

## What is one Governed AI System

A Governed AI System is counted by **model name**:

- Every version of a model is the same system. Registering `credit_scoring` version `2.1.0` when
  `credit_scoring` `2.0.0` exists never counts as a new system.
- **Archived** models don't count.
- **Sample-data** models loaded from **Settings → Sample data & tour** don't count.

When all of your plan's systems are in use, registering a model under a **new** name (or restoring
an archived one) is refused with `403 Forbidden`:

```json
{
  "error": "HTTPException",
  "detail": "Your plan includes 1 Governed AI System, and all are in use. Archive one you no longer govern, or contact us to govern more.",
  "status_code": 403
}
```

To free a slot, archive a system you no longer govern: open it in the
[Model Register](../user-guide/model-register.md), go to **System settings → Archive**, or call
`client.models.archive(model_id)`. Archived systems keep their history and can be restored later.

## When you reach the limit

### Free plan: `429 QuotaExceeded`

The Free plan's 2,500 events per month is a hard cap. A request that would record new activity once
the allowance is used is refused:

```json
{
  "error": "QuotaExceeded",
  "detail": "This organization has used its 2,500 Governance Events for the month. Upgrade to Business Cloud to keep recording.",
  "status_code": 429
}
```

A batch larger than what's left is **refused whole**, so nothing is half-recorded:

```json
{
  "error": "QuotaExceeded",
  "detail": "This request would record 500 Governance Events, but only 20 of this month's 2,500 remain. Send fewer records, or upgrade to Business Cloud.",
  "status_code": 429
}
```

Everything already recorded stays available, and free activity keeps working. Recording resumes at
the start of the next month.

### Business Cloud: billed as overage

Business Cloud is never blocked for usage. Events past the 100,000 included each month are recorded
normally and billed at $5 per 10,000.

### Rate limits are different

A `429` with `"error": "RateLimitExceeded"` is the per-client **rate limit** (100 requests per 60
seconds by default), not your monthly allowance. It comes with `X-RateLimit-Limit`,
`X-RateLimit-Remaining`, `X-RateLimit-Reset` and `Retry-After` headers. Wait for `Retry-After`
and try again. The Python SDK raises `RateLimitError` with the server's message for both kinds of
`429`.

## See your usage

**Settings → Subscription & usage** shows how many Governance Events your organization has used
this month out of its allowance. On Business Cloud, usage past the allowance is shown as **billed
as overage**.
