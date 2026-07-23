# Troubleshooting

Common issues you might run into while using WhiteBoxXAI, and how to resolve them. If
you're still stuck after trying these steps, reach out — see [Getting help](#getting-help).

## SDK issues

### "No module named 'whiteboxxai'"

The SDK isn't installed in the environment you're running. Install it:

```bash
pip install whitebox-xai-sdk
```

Then verify it's available:

```python
import whiteboxxai
print(whiteboxxai.__version__)
```

If you have multiple Python environments, make sure you're installing into the same one
that runs your code (check with `import sys; print(sys.executable)`).

### "cannot import name 'WhiteBoxXAI'"

You may have an outdated version. Upgrade to the latest:

```bash
pip install --upgrade whitebox-xai-sdk
```

The package was renamed from the earlier `whiteboxai` naming to `whiteboxxai` — make sure
your imports use `from whiteboxxai import WhiteBoxXAI`.

## Authentication

### "401 Unauthorized: Invalid API key"

- Double-check the key was copied correctly, with no leading/trailing spaces or newlines.
- Confirm you're using an active key: in the dashboard, go to **Profile → API Keys**.
- If in doubt, generate a new key and update your integration. Remember that a key is shown
  in full only once, at creation time.

```python
import os
print(os.getenv("WHITEBOXXAI_API_KEY"))  # verify the key your app is actually using
```

### "403 Forbidden: Insufficient permissions"

Your account or API key doesn't have permission for that action. Check your role under
**Settings → Team**, and note that **demo** accounts are read-only — they can view data but
can't create, edit, or delete. See [Plans & Limits](../get-started/plans.md).

## Predictions aren't appearing

Your SDK calls succeed but you don't see predictions in the dashboard. Work through these
in order:

1. **Check the model ID.** Make sure you're logging to the right model:

    ```python
    for m in client.models.list():
        print(m.id, m.name)
    ```

2. **Check your sampling rate.** If you configured sampling, some predictions are logged
   intentionally. Set the rate to `1.0` to log every prediction while debugging:

    ```python
    monitor = ModelMonitor(client=client, model_id=model_id, sampling_rate=1.0)
    ```

3. **Allow time for async delivery.** If you log asynchronously or with buffering,
   predictions are sent in the background. Flush the buffer, or give it a moment before
   refreshing the dashboard.

4. **Enable debug logging** to confirm requests are being sent:

    ```python
    import logging
    logging.basicConfig(level=logging.DEBUG)
    ```

If you're on the **Free** plan and have hit your monthly API-call limit, new requests are
paused until your next cycle — see [Plans & Limits](../get-started/plans.md).

## Two-factor authentication

### My 6-digit code isn't accepted

- **Time drift is the most common cause.** Authenticator codes depend on your device clock.
  Enable automatic date/time on your phone, or turn on "time correction" in your
  authenticator app's settings.
- **Codes expire every 30 seconds and can only be used once.** Wait for a fresh code and
  enter the one currently shown.

### I've lost my authenticator app

Use one of your saved **backup codes** at the 2FA prompt. Each backup code works once. After
logging in, go to **Profile → Security** to set up a new authenticator and regenerate backup
codes. Full details: [Two-Factor Authentication](../account/two-factor-authentication.md).

### I've lost my app *and* my backup codes

Contact **[support@whiteboxxai.com](mailto:support@whiteboxxai.com)**. Account recovery
requires identity verification and can take 24–48 hours.

## Understanding API errors

When an API request fails, the response body includes an error `code` and `message`. The
HTTP status tells you the category:

| Status | Meaning | What to check |
| --- | --- | --- |
| 400 | Bad Request | Invalid input or malformed request body |
| 401 | Unauthorized | Missing or invalid API key |
| 403 | Forbidden | Insufficient permissions, or a read-only (demo) account |
| 404 | Not Found | The resource ID doesn't exist |
| 409 | Conflict | The resource already exists |
| 422 | Validation Error | A field doesn't match the expected format |
| 429 | Too Many Requests | Rate limit or monthly quota exceeded |
| 500 | Server Error | A problem on our side — retry, then contact support |

A 422 response tells you exactly which field is wrong:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input",
    "details": { "field": "model_type", "error": "Must be one of: classification, regression" }
  }
}
```

### Rate limits and quotas (429)

A `429` means you've sent too many requests too quickly, or reached your plan's monthly
allowance. To avoid it:

- **Batch** your prediction logging instead of one call per prediction.
- **Sample** high-volume models rather than logging every prediction.
- Retry with a short backoff after a brief pause.

See [Plans & Limits](../get-started/plans.md) for the allowance on each plan.

## Getting help

Before reaching out, it helps to gather:

- The exact error message and the API `code` (if any)
- Steps to reproduce the issue
- Your SDK version (`whiteboxxai.__version__`)

Then contact us:

- **Documentation:** [docs.whiteboxxai.com](https://docs.whiteboxxai.com)
- **Community forum:** [community.whiteboxxai.com](https://community.whiteboxxai.com)
- **Email support:** [support@whiteboxxai.com](mailto:support@whiteboxxai.com)
