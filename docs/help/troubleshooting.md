---
description: Fixes for common WhiteBoxXAI problems — SDK install and import errors, 401/403/404/409/422/429 responses, plan limits, predictions or explanations not appearing, MCP server errors, governance votes refused, and two-factor issues.
---

# Troubleshooting

Find the error you're seeing, then work through the steps. If you're still stuck, see
[Getting help](#getting-help).

## Reading an error

Every API error uses the same envelope, and the SDK raises a matching exception:

```json
{"error": "HTTPException", "detail": "Model not found", "status_code": 404}
```

| Status | `error` | SDK exception | Usual cause |
| --- | --- | --- | --- |
| `400` | `HTTPException` | `APIError` | The request can't be carried out as sent, for example an invalid YAML policy |
| `401` | `HTTPException` | `AuthenticationError` | Missing, mistyped, revoked or expired key |
| `403` | `HTTPException` | `APIError` | Role, plan limit, or an action that needs a person's own session |
| `404` | `HTTPException` | `APIError` | The id doesn't exist in **your** organization |
| `409` | `HTTPException` | `APIError` | Duplicate name, or the record changed since you loaded it |
| `422` | `ValidationError` | `ValidationError` | A field is missing or the wrong type; `detail` names each one |
| `429` | `QuotaExceeded` or `RateLimitExceeded` | `RateLimitError` | Monthly allowance used up, or too many requests |
| `5xx` | | `ServerError` | A problem on our side; the SDK retries, then raises |

Every response has an `X-Request-ID` header. Include it when you contact support.

## SDK

### `No module named 'whiteboxxai'`

The SDK isn't installed in the environment running your code:

```bash
pip install whitebox-xai-sdk
python -c "import whiteboxxai, sys; print(whiteboxxai.__version__, sys.executable)"
```

The package is `whitebox-xai-sdk`; the import is `whiteboxxai`.

### `ValueError` about a missing API key, or `TypeError: ... 'api_key'`

`WhiteBoxXAI()` reads the key from `WHITEBOX_API_KEY` (and the address from `WHITEBOX_BASE_URL`) when
you don't pass them. The error means the variable isn't set in the process running your code, so
check it there, not only in your shell. You can also pass the key explicitly:

```python
import os
from whiteboxxai import WhiteBoxXAI

client = WhiteBoxXAI(api_key=os.environ["WHITEBOX_API_KEY"])
```

Older SDK versions needed the key passed in, so upgrade with `pip install -U whitebox-xai-sdk` if you
still see the `TypeError`. The older names `WHITEBOXXAI_API_KEY` and `WHITEBOXXAI_BASE_URL` are also read.

### `ValueError` from an integration helper's `register_from_model()`

The helper couldn't work out the model's feature names, or you passed a `model_type` the API doesn't
accept. Pass `features=[...]` with the input column names, and use one of `xgboost`, `random_forest`,
`neural_network`, `linear_model`, `llm` or `custom`. A task name such as `"classification"` is kept as
metadata and the framework's default type is used. See
[Python SDK → Framework integrations](../sdk/index.md#framework-integrations).

If you hit an `AttributeError` or a refused registration from one of these helpers, you are on an older
SDK than 1.3.0. Upgrade, or register with `client.models.register(...)` and pass `model_id=` to the
monitor.

## Authentication

### `401 Unauthorized`

- Check the key was copied whole, with no spaces or line breaks. Keys start with `wbx_live_`.
- Check `WHITEBOX_API_KEY` is set in the process that makes the request, not only in your shell.
- In **Settings → API keys**, check the key hasn't been revoked or passed its expiry. A key also stops
  working if the administrator who created it is deactivated.
- Using a signed-in session token rather than a key? It expires after about 30 minutes. Use an
  [API key](../account/api-keys.md) for anything unattended.

### `403 Forbidden`

- **Your role.** Some actions are administrator-only, such as creating API keys, approving evidence,
  deciding control applicability, or editing boards and scales. Check your role in **Settings → Users
  & roles**.
- **Your plan's Governed AI Systems are all in use.** Registering a model under a new name, or
  restoring an archived one, returns *"Your plan includes 1 Governed AI System, and all are in use."*
  Archive a system you no longer govern, or register a new **version** of an existing one. See
  [Governance Events](../get-started/governance-events.md#what-is-one-governed-ai-system).
- **Model diff on the Free plan.** Model-diff reviews are included on Business Cloud and Enterprise
  Edition.
- **Voting or finalizing a review with an API key.** Votes need a person's own session; see
  [Votes are refused](#votes-are-refused).
- **Demo accounts are read-only.**

### `404 Not Found`

A `404` on an id means the record doesn't exist **in your organization**: records in other
organizations look exactly like records that don't exist. Check you're using the right id and the right
API key. Don't retry with other credentials.

## Limits

### `429` with `QuotaExceeded`

Your Free plan has used its 2,500 Governance Events for the month, or a batch is larger than what's
left (a batch is refused whole). Recording resumes next month; reads, configuration, reports and the
dashboard keep working. To use events more slowly, sample high-volume models and group run records with
`governance_run()`. See [Governance Events](../get-started/governance-events.md).

### `429` with `RateLimitExceeded`

Too many requests in the current window (100 per 60 seconds by default). Wait for the number of seconds
in `Retry-After`. The SDK does this for waits of 60 seconds or less. Use batch endpoints for volume.

## Data doesn't appear

### Predictions aren't showing

1. **Check the model id** you're logging to:

    ```python
    for m in client.models.list(search="credit"):
        print(m["id"], m["name"], m["version"])
    print(client.predictions.get_recent(model_id))
    ```

2. **Check sampling.** With `ModelMonitor(sampling_rate=...)` below 1.0, some predictions are skipped
   on purpose.
3. **Flush buffers.** With `buffer_size` set, call `monitor.flush()` or use the monitor as a context
   manager.
4. **Check offline mode.** With `enable_offline=True`, records may be queued locally:
   `client.get_offline_status()`.
5. **Check for a `429`.** On the Free plan, the monthly allowance may be used up.

### Registering a model returns `422`

The API needs `name`, `version`, `model_type`, `features` and `baseline_metrics`. `model_type` must be
one of `xgboost`, `random_forest`, `neural_network`, `linear_model`, `llm` or `custom`; task names like
`classification` aren't accepted. `baseline_metrics={}` is valid when you haven't measured any.

### An explanation is `failed`

Explanations run the model itself, so WhiteBoxXAI needs the model file. If the message says no stored
model artifact is available, the model was registered with metadata only. Register it through
[GitHub](../integrations/github.md#configuration-file) with `storage_mode: full`.

### A drift report or fairness audit says "not enough data"

Results are computed from what was logged. With little history, WhiteBoxXAI says it can't judge yet
rather than estimating. A fairness audit also needs the true outcomes and group membership you supply.

### A report type says **Needs data**

The card gives the reason, for example no bias audit yet for that system, or no applicability decisions
for that framework. See [Report types reference](../user-guide/report-engine.md).

### An alert rule never fires

Rules are evaluated when metrics are logged, drift reports are created and bias audits complete. Check
the rule's `metric_name` matches the name the value arrives under, the rule is active and for the right
model, and that it isn't throttled. Test it with `client.alerts.evaluate_rule(rule_id, metric_values={...})`.
See [When rules are evaluated](../user-guide/needs-attention.md#when-rules-are-evaluated).

## Governance

### Votes are refused

- **`403` asking for two-factor authentication.** Turn on two-factor in **Settings → Profile &
  security**. Every vote and finalization also needs a current authenticator or backup code.
- **`403` with an API key.** Organization API keys can't vote or finalize. Vote from your own
  signed-in session.
- **`409` "This request changed after you opened it."** Reload the request, review the current version,
  and vote again.
- **Rejected outright.** You raised the request, and submitters can't vote on their own requests.
- **`429` "Too many incorrect codes."** Wait, then try again with a fresh code.

## MCP server

### `whiteboxxai-mcp: no credentials configured` (exit code 2)

Set `WHITEBOXXAI_MCP_API_KEY` in the environment your MCP client starts the server with, usually the
`env` block of its config, not only in your shell. The local server accepts only an API key.

### `401` on the hosted endpoint

Every request to `https://api.whiteboxxai.com/mcp/` needs `Authorization: Bearer <key or token>`. A
personal sign-in token expires after about 30 minutes; reconnect, or use an API key.

### The client doesn't show the tools

Restart the client after editing its config, and check `whiteboxxai-mcp` is on the `PATH` the client
uses (or give the full path to the command). See [MCP Server](../integrations/mcp.md).

## Two-factor authentication

### My 6-digit code isn't accepted

- Device clock drift is the most common cause. Turn on automatic date and time on your phone.
- Codes change every 30 seconds and work once. Wait for a fresh one.

### I've lost my authenticator app

Sign in with one of your saved **backup codes**; each works once. Then set up a new authenticator and
regenerate backup codes in **Settings → Profile & security**. See
[Two-Factor Authentication](../account/two-factor-authentication.md).

### I've lost my app and my backup codes

Email **[support@whiteboxxai.com](mailto:support@whiteboxxai.com)**. Account recovery requires identity
verification.

## Getting help

Gather the error message, the `X-Request-ID`, what you did, and your SDK version
(`pip show whitebox-xai-sdk`), then use **Support** in the dashboard sidebar or email
**[support@whiteboxxai.com](mailto:support@whiteboxxai.com)**. See [Getting support](support.md).
