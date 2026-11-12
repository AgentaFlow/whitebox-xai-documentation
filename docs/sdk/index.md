---
description: The WhiteBoxXAI Python SDK (whitebox-xai-sdk 1.3.0) — install, configure, register models, log predictions and LLM calls, group runs into Governance Events, explanations including DiCE, drift, fairness, prompts, compliance, reports, governance, async, offline mode, privacy helpers and error handling.
---

# Python SDK

The `whitebox-xai-sdk` package is the Python client for WhiteBoxXAI. It wraps the REST API with typed
methods, retries transient failures, can queue records while offline, and has drop-in integrations
for common ML and LLM frameworks.

- **Package:** `whitebox-xai-sdk` 1.3.0 · **Import:** `whiteboxxai` · **Python:** 3.9 or later ·
  **License:** MIT
- **Source and examples:** [github.com/AgentaFlow/whitebox-python-sdk](https://github.com/AgentaFlow/whitebox-python-sdk)

## Install

```bash
pip install whitebox-xai-sdk
```

Framework integrations are optional extras:

| Extra | Adds |
| --- | --- |
| `whitebox-xai-sdk[sklearn]` | scikit-learn |
| `whitebox-xai-sdk[pytorch]` | PyTorch |
| `whitebox-xai-sdk[tensorflow]` | TensorFlow / Keras |
| `whitebox-xai-sdk[xgboost]`, `[lightgbm]` | XGBoost, LightGBM |
| `whitebox-xai-sdk[huggingface]` | Hugging Face Transformers |
| `whitebox-xai-sdk[langchain]` | LangChain / LangGraph |
| `whitebox-xai-sdk[all]` | All of the above |

Check the installed version with `python -c "import whiteboxxai; print(whiteboxxai.__version__)"`.

## Configure the client

```python
import os

from whiteboxxai import WhiteBoxXAI

client = WhiteBoxXAI(api_key=os.environ["WHITEBOX_API_KEY"])
```

| Parameter | Default | Purpose |
| --- | --- | --- |
| `api_key` | required | An [API key](../account/api-keys.md) (`wbx_live_...`) |
| `base_url` | `https://api.whiteboxxai.com` | Your API address. Business Cloud tenants use the address they were given. Also read from `WHITEBOX_BASE_URL`. |
| `timeout` | `30` | Request timeout, in seconds |
| `max_retries` | `3` | Retries for connection errors, `5xx` responses and `429`s |
| `enable_offline` | `False` | Queue records locally when the API can't be reached. See [Offline Mode](offline-mode.md). |

The client can be used as a context manager (`with WhiteBoxXAI(...) as client:`, or `async with`), which
closes its connections on exit. Otherwise call `client.close()`.

### Secure configuration

- Read the key from the environment (`WHITEBOX_API_KEY`) or a secrets manager. Never hard-code it,
  commit it, put it in a notebook you share, or log it.
- Use one key per application or pipeline, so you can revoke one without breaking the rest.
- Keys act with the permissions of the administrator who created them. See
  [API Keys](../account/api-keys.md#what-a-key-can-do).

## What the client exposes

Every method returns plain Python dicts and lists parsed from the API's JSON. Ids are strings (UUIDs).
Each method has an async twin prefixed with `a`, such as `client.models.aregister(...)`.

| Resource | Use it for | Common methods |
| --- | --- | --- |
| `client.models` | Register and manage AI systems and their versions | `register`, `get`, `list`, `update`, `get_versions`, `get_latest`, `update_baseline`, `archive`, `restore`, `compare_versions`, `version_changelog`, `list_git_versions` |
| `client.predictions` | Log and read predictions | `log`, `log_batch`, `get`, `query`, `get_recent`, `get_stats` |
| `client.explanations` | SHAP, LIME and DiCE explanations | `generate`, `generate_async`, `generate_bulk`, `get`, `get_by_prediction`, `set_config`, `compare` |
| `client.drift` | Drift checks and reports | `detect`, `create_report`, `get_reports`, `get_report`, `get_trend` |
| `client.fairness` | Bias audits | `audit`, `get_audit`, `list_audits`, `get_bias_history` |
| `client.metrics` | Performance metrics you report | `create`, `create_batch`, `calculate_classification`, `calculate_regression`, `summary`, `trend`, `timeseries` |
| `client.alerts` | Alert rules and alert instances | `create`, `list`, `evaluate_rule`, `list_instances`, `acknowledge`, `resolve` |
| `client.alert_intelligence`, `client.alert_management` | Alert prioritisation, correlation, root cause, comments, history | |
| `client.llm` | LLM call logging and usage | `log_call`, `log_calls_batch`, `query_logs`, `get_stats`, `usage_stats`, `cost_breakdown` |
| `client.rag` | RAG retrievals and evaluations | `log_retrieval`, `create_evaluation`, `get_stats`, `metrics_precision` |
| `client.safety` | Toxicity, PII and harmful-content scans | `analyze`, `analyze_batch`, thresholds |
| `client.llm_xai` | LLM explainability | `attention`, `token_importance`, `prompt_sensitivity`, `counterfactuals`, `debug_prompt`, `batch_analyze`; `rag_attribution` and `get_unified_explanation` in the next SDK release |
| `client.prompts` | Governed prompt versions | `create_template`, `create_version`, `advance_stage`, `deploy`, `rollback`, `compute_drift` |
| `client.agent_workflows` | Multi-agent workflow tracing | `create_and_start`, `register_agent`, `complete`, `analytics` |
| `client.risk_register` | The AI Risk Register | `list`, `get`, `portfolio` |
| `client.governance` | Review boards and requests | `list_boards`, `list_review_requests`, `get_review_request`, `submit_decision` |
| `client.compliance` | Framework catalogs and ISO/IEC 42001 readiness | `list_frameworks`, `get_framework_controls`, `iso42001_score`, `iso42001_gaps` |
| `client.reports` | Report types | `list`, `preview`, `generate` |

## Register a model

```python
model = client.models.register(
    name="credit_scoring",
    model_type="xgboost",
    framework="xgboost",
    version="1.0.0",
    features=["income", "utilisation", "tenure_months"],
    baseline_metrics={"accuracy": 0.94},
    auto_detect_git=True,
)
model_id = model["id"]
```

- `model_type` is one of `xgboost`, `random_forest`, `neural_network`, `linear_model`, `llm` or
  `custom`.
- `version`, `features` and `baseline_metrics` are required by the API. Only pass baseline metrics
  you actually measured; `{}` is valid.
- Registering the same name and version again returns the existing model, so it's safe on every
  start-up. A new version string creates a new version of the same
  [Governed AI System](../get-started/governance-events.md#what-is-one-governed-ai-system).
- `auto_detect_git=True` records the commit, branch and repository the model came from;
  `require_clean_git=True` refuses to record a dirty working tree. See
  [Git context tracking](../integrations/cli-git-tracking.md).

Other optional fields include `description`, `target_variable`, `tags`, `fairness_attributes` and
`protected_groups`.

## Log predictions

```python
client.predictions.log(
    model_id=model_id,
    input_data={"income": 52000, "utilisation": 0.31, "tenure_months": 18},
    output_data={"prediction": 0, "probability": 0.12},
    latency_ms=4.2,
    metadata={"customer_ref": "c-1042"},
)

client.predictions.log_batch(
    model_id,
    predictions=[{"input_data": {...}, "output_data": {...}}, ...],   # up to 1,000
)
```

Each record is one [Governance Event](../get-started/governance-events.md). Batching costs the same as
logging one at a time; group related records into runs to share events.

## ModelMonitor and decorators

`ModelMonitor` wraps registration and logging, with sampling and buffering:

```python
from whiteboxxai import ModelMonitor, monitor_model

monitor = ModelMonitor(
    client,
    model_name="credit_scoring",
    sampling_rate=0.1,      # log 10% of predictions
    buffer_size=100,        # send in batches of 100
)
monitor.register_model(
    name="credit_scoring", model_type="xgboost", framework="xgboost", version="1.0.0",
    features=["income", "utilisation", "tenure_months"], baseline_metrics={},
)

@monitor_model(monitor, input_keys=["features"])
def predict(features):
    return model.predict([features])[0]
```

With `buffer_size` set, call `monitor.flush()` before the process exits, or use the monitor as a
context manager (`with ModelMonitor(...) as monitor:`), which flushes for you.

`monitor_prediction` takes custom extractors when your function's arguments or return value don't map
directly onto inputs and outputs:

```python
from whiteboxxai import monitor_prediction

@monitor_prediction(
    monitor,
    input_extractor=lambda args, kwargs: {"features": kwargs["data"]},
    output_extractor=lambda result: result["prediction"],
)
def predict(data):
    return {"prediction": 0.85, "confidence": 0.92}
```

## Framework integrations

Import from `whiteboxxai.integrations.<module>`:

| Framework | Module | Classes |
| --- | --- | --- |
| scikit-learn | `sklearn` | `SklearnMonitor`, `SklearnWrapper` |
| XGBoost / LightGBM | `boosting` | `XGBoostMonitor`, `LightGBMMonitor` |
| PyTorch | `pytorch` | `TorchMonitor`, `TorchWrapper` |
| TensorFlow / Keras | `tensorflow` | `KerasMonitor`, `WhiteBoxXAICallback` |
| Hugging Face Transformers | `transformers` | `TransformersMonitor` |
| LangChain | `langchain` | `LangChainMonitor`, `WhiteBoxXAICallbackHandler` |
| LangGraph and LangChain multi-agent systems | none: use `client.agent_workflows` | See [LangChain Multi-Agent](../integrations/langchain-multi-agent.md) |
| CrewAI | `crewai_monitor` | `CrewAIMonitor`, `monitor_crew` |

```python
from whiteboxxai.integrations.sklearn import SklearnMonitor, SklearnWrapper

monitor = SklearnMonitor(client, model=clf)
monitor.register_from_model(
    name="churn", model_type="random_forest", version="2.0.0",
    features=list(X.columns), baseline_metrics={},
)
wrapped = SklearnWrapper(clf, monitor)
wrapped.predict(X_test)        # logged automatically
```

!!! warning "Known issues in SDK 1.3.0"
    Some integration helpers can't register models or workflows against the current API:
    `XGBoostMonitor`/`LightGBMMonitor.register_from_model`, `TransformersMonitor.register_from_model`,
    `KerasMonitor.register_saved_model`, `LangChainMonitor.register_application` and
    `create_callback_handler`, and the `langchain_agents` helpers. Register with
    `client.models.register(...)` and pass `model_id=` to the monitor, as each framework guide shows.

Per-framework guides: [TensorFlow](../integrations/tensorflow.md),
[Hugging Face](../integrations/huggingface.md), [XGBoost & LightGBM](../integrations/boosting.md),
[LangChain](../integrations/langchain.md), [CrewAI](../integrations/crewai.md).

## LLM, RAG and agents

```python
client.llm.log_call(
    provider="anthropic",
    model_name="claude-sonnet-4-5",
    prompt=prompt,
    completion=answer,
    latency_ms=910.0,
    prompt_tokens=388,
    completion_tokens=121,
    prompt_version_id=version_id,      # optional: link to a governed prompt version
)
client.safety.analyze(content=answer, content_type="completion")
client.rag.log_retrieval(query=question, results=docs, top_k=5, retrieval_method="vector")
```

`provider` is any string, so you can log calls to any LLM provider. See
[LLM monitoring](llm-monitoring.md) for provider recipes, and
[Multi-Agent Monitoring](multi-agent-monitoring.md) for agent workflows.

## Group a run: `governance_run()`

Records from one end-to-end AI run can share
[Governance Events](../get-started/governance-events.md#grouped-runs-one-event-per-100-records): every
100 predictions, LLM calls, retrievals or safety scans sent inside `governance_run()` cost one event.

```python
from whiteboxxai import governance_run

with governance_run() as run_id:            # or governance_run("ticket-4821")
    client.llm.log_call(...)
    client.rag.log_retrieval(...)
    client.safety.analyze(...)
```

The block sets the `X-WhiteBox-Run-Id` header on every SDK request made inside it. A run id is up to
128 letters, digits, `.`, `_`, `:` or `-`; anything else is ignored by the server, and those records
are counted individually. The LangChain callback handler groups each root chain run automatically.

## Explanations

```python
explanation = client.explanations.generate(
    model_id,
    instance={"income": 52000, "utilisation": 0.31, "tenure_months": 18},
    method="shap",          # "shap", "lime" or "dice"; omit for the model's default
    num_features=10,
)
```

!!! note "Explanations run your model"
    SHAP, LIME and DiCE explanations are computed by running the model itself, so WhiteBoxXAI needs
    the model file. Register the model through [GitHub](../integrations/github.md#configuration-file)
    with `storage_mode: full`; scikit-learn-style pickle and joblib files are supported. For a model
    registered with metadata only, the explanation is recorded as **failed**, with a message saying no
    stored model artifact is available. Drift, fairness audits and reports don't need the model file.

The result is a dict with feature weights and a base value. For counterfactuals, use
`method="dice"` with `total_cfs` (default 3), `desired_class` (default `"opposite"`) and
`features_to_vary` (default `"all"`, or a list of feature names).

For many instances, `generate_bulk(model_id, instances=[...])` (up to 100) and
`generate_async(model_id, instance=...)` return immediately with ids. Poll `get(explanation_id)` until
its status isn't `pending`. Set per-model defaults with
`set_config(model_id, enabled=True, default_method="shap", auto_explain=False)`.

See [Explainability Engine](../user-guide/features.md#explainability-engine) for how values are computed.

## Drift and fairness

```python
quick = client.drift.detect(model_id, window_size=1000)          # not stored
report = client.drift.create_report(model_id, window_size=1000)  # stored; cite report["id"]
trend = client.drift.get_trend(model_id, days=30)

audit = client.fairness.audit(
    model_id,
    sensitive_attributes=["gender"],
    y_true=y_true,
    y_pred=y_pred,
    group_data={"gender": genders},
)
```

A fairness audit needs true labels, predicted labels and group membership that you supply; WhiteBoxXAI
doesn't infer protected attributes. There's no endpoint for attaching an actual outcome to an earlier
prediction, so join outcomes from your own records.

## Prompt versions

```python
template = client.prompts.create_template(name="support-bot")
v = client.prompts.create_version(
    template["id"],
    user_message_template="Answer the customer: {question}",
    system_message="You are a careful support assistant.",
    model_name="gpt-4o",
    temperature=0.2,
    version_tag="v3",
    commit_hash="9f1c2ab",
)
client.prompts.advance_stage(template["id"], v["id"], target_stage="verify")
client.prompts.deploy(template["id"], v["id"])     # refused without a passed verification
client.prompts.rollback(template["id"], target_version_id=previous_id, reason="tone regression")
```

Don't pass `force=True` to `deploy` to get around a failed verification. See
[Evaluations & Prompt Versions](../user-guide/evaluations.md).

## Compliance, reports and governance

```python
client.compliance.list_frameworks()                  # keys: iso_42001, eu_ai_act, gdpr, ccpa, nist_ai_rmf
client.compliance.get_framework_controls("iso_42001")
client.compliance.iso42001_gaps(severity_filter="high")

client.reports.list()
client.reports.generate(report_type_id="model_card", model_id=model_id, format="pdf")

client.risk_register.portfolio()
client.governance.list_review_requests(status="under_review")
client.governance.get_review_request(request_id)     # includes evidence_snapshot_hash
```

`client.governance.submit_decision(...)` exists for a person voting from their own session. It needs
`evidence_snapshot_hash` and a current authenticator code (`totp_code`), and is refused when called with
an organization API key. See [Reviews & Decisions](../user-guide/governance.md#vote-with-an-authenticator-code).

## Compare model versions

```python
diff = client.models.compare_versions(version1_id, version2_id)
changelog = client.models.version_changelog(version1_id, version2_id)   # markdown
client.models.list_git_versions(model_id)
```

Available on Business Cloud and Enterprise Edition. See [GitHub Diff Reviews](../integrations/github-diff.md).

## Async

Every resource method has an async twin prefixed with `a`:

```python
import asyncio
import os

from whiteboxxai import WhiteBoxXAI

async def main():
    async with WhiteBoxXAI(api_key=os.environ["WHITEBOX_API_KEY"]) as client:
        await asyncio.gather(*[
            client.predictions.alog(model_id, input_data=x, output_data=y) for x, y in batch
        ])

asyncio.run(main())
```

## Offline mode

With `enable_offline=True`, records that can't be sent are queued on disk and synced when the API is
reachable again. `client.get_offline_status()` reports the queue, and `client.sync_offline_queue()`
syncs immediately. Records are only counted, and only appear in the dashboard, once they've been
delivered. See [Offline Mode](offline-mode.md).

## Privacy helpers

`whiteboxxai.privacy` detects and masks common personal data before you log it:

```python
from whiteboxxai.privacy import DataMasker, detect_pii, mask_pii

detect_pii("Contact jane@example.com or 555-123-4567")   # email and phone findings
mask_pii("Contact jane@example.com")                       # "Contact ****************"

masked = DataMasker().mask({"email": "jane@example.com", "amount": 100.0})
```

Detected types include email addresses, phone numbers, US Social Security numbers, credit card
numbers (Luhn-checked) and IP addresses. Masking is a safety net; the better practice is not to log
identifiers the model doesn't use.

## Errors

All SDK exceptions inherit from `whiteboxxai.exceptions.WhiteBoxXAIError`:

| Exception | When |
| --- | --- |
| `AuthenticationError` | `401`: missing, wrong, revoked or expired key |
| `ValidationError` | `422`: the request didn't match the API's schema; `.fields` has details |
| `RateLimitError` | `429`: the rate limit or, on the Free plan, the monthly Governance Events allowance; `.retry_after` is set when the server sent `Retry-After` |
| `APIError` | Any other `4xx`, including `403` and `404` (the id doesn't exist in your organization). Check `.status_code`; `.request_id` helps support trace a request. |
| `ServerError`, `APIConnectionError` | `5xx` responses and network failures |
| `ConfigurationError`, `IntegrationError` | Client misconfiguration, or a framework integration problem |

The client retries connection errors, `5xx` responses and `429`s up to `max_retries` times with
exponential back-off, honouring `Retry-After` when it's 60 seconds or less. A longer wait is raised
straight away as `RateLimitError`, so your code decides whether to wait.

```python
from whiteboxxai.exceptions import APIError, RateLimitError

try:
    client.predictions.log(model_id=model_id, input_data=x, output_data=y)
except RateLimitError as err:
    print("Throttled or over quota:", err, "retry after", err.retry_after)
except APIError as err:
    if err.status_code == 404:
        print("That model isn't in this organization")
    else:
        raise
```

A `403` when registering a model means your plan's Governed AI Systems are all in use. See
[Troubleshooting](../help/troubleshooting.md).

## Next steps

- [API Reference](api-reference.md): the REST endpoints behind these methods
- [LLM monitoring](llm-monitoring.md) and [Multi-Agent Monitoring](multi-agent-monitoring.md)
- [Integration examples](../integrations/examples.md)
- [MCP Server](../integrations/mcp.md), for AI assistants rather than application code
