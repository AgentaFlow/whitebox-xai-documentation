# API Keys

API keys are the credential your code uses to talk to WhiteBoxXAI. They're scoped,
individually revocable, and don't expire on a fixed schedule — which makes them the right
choice for the [SDK](../sdk/index.md), CI/CD pipelines, the [MCP
server](../integrations/mcp.md), and anything else running unattended.

Keys look like this:

```text
wbx_live_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

## Create a key

### In the dashboard

Go to **Profile → API Keys → Generate New**, give the key a name, and copy it.

### Via the API

Key creation and revocation are **admin-only**, because an API key is an organization-wide
credential rather than a personal one.

```bash
curl -X POST https://api.whiteboxxai.com/api/v1/api-keys \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "CI pipeline — nightly drift job",
    "scopes": ["read", "write"],
    "expires_in_days": 365
  }'
```

| Field | Notes |
| --- | --- |
| `name` | Required, max 255 characters. Make it descriptive — it's how you'll identify the key later. |
| `scopes` | Optional list of coarse scopes, e.g. `["read", "write"]`. Defaults to none. |
| `expires_in_days` | Optional, 1–3650. Omit for a key that never expires. |

The `201` response is the **only** time you see the key material:

```json
{
  "id": "a1b2c3d4-...",
  "name": "CI pipeline — nightly drift job",
  "key_prefix": "wbx_live_a1b2c3d4...",
  "scopes": ["read", "write"],
  "expires_at": "2027-07-29T00:00:00Z",
  "last_used_at": null,
  "revoked_at": null,
  "created_at": "2026-07-29T00:00:00Z",
  "raw_key": "wbx_live_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
}
```

!!! warning "`raw_key` is shown exactly once"
    Store it immediately in your secrets manager. WhiteBoxXAI keeps only a hash and the
    prefix — the full key cannot be retrieved again. If you lose it, revoke the key and
    issue a new one.

## Use a key

Send it as a bearer token:

```bash
curl https://api.whiteboxxai.com/api/v1/models \
  -H "Authorization: Bearer wbx_live_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

With the Python SDK:

```python
from whiteboxxai import WhiteBoxXAI

client = WhiteBoxXAI(api_key="wbx_live_...")
```

Or, preferably, from the environment:

```bash
export WHITEBOXXAI_API_KEY="wbx_live_..."
```

```python
client = WhiteBoxXAI()  # reads WHITEBOXXAI_API_KEY
```

## List keys

```bash
GET /api/v1/api-keys
```

Returns every key in your organization with its metadata — never the key material, only the
`key_prefix` for identification. `last_used_at` tells you whether a key is still in service,
which is what you want before revoking one.

## Revoke a key

```bash
DELETE /api/v1/api-keys/{key_id}
```

Returns `204`. Revocation is immediate — any process still using that key starts getting
`401` on its next request. Revoke a key as soon as you suspect exposure; unlike a password,
there's nothing else to rotate.

## API keys vs. login tokens

The `Authorization: Bearer` header accepts either an API key or a login JWT from
`POST /api/v1/auth/login`. They are not equivalent:

| | API key | Login JWT |
| --- | --- | --- |
| Lifetime | Until you revoke it (or its optional expiry) | ~30 minutes |
| Scope | Coarse scopes you choose | The full privileges of the human account |
| Revocable individually | Yes | No — you'd rotate the account's credentials |
| Good for | SDK, CI/CD, pipelines, MCP server, webhooks | Quick interactive scripts, the dashboard |

Use a JWT for a throwaway script if it's convenient. Use an API key for anything that runs
without a person watching it — a 30-minute token in a nightly job is a job that fails at
minute 31.

## Keys and two-factor authentication

[Two-factor authentication](two-factor-authentication.md) protects interactive dashboard
logins. It does **not** affect API keys — a key authenticates on its own, with no 2FA
challenge. Enabling 2FA won't break your existing integrations, and no code changes are
needed.

The practical consequence: an API key is a standalone credential, so treat it with the same
care as a password. Keep keys out of source control, out of notebooks you share, and out of
log output.

## Good practice

- **One key per consumer.** A separate key for each pipeline, environment, and service means
  you can revoke one without taking down everything else.
- **Name keys after where they run.** "CI pipeline — nightly drift job" beats "key 3" when
  you're deciding six months later whether it's safe to revoke.
- **Set an expiry for temporary access.** Contractors, spikes, and one-off migrations should
  get a key with `expires_in_days`.
- **Check `last_used_at` before revoking.** And revoke anything that hasn't been used in a
  while — an unused key is pure risk.
- **Store keys in a secrets manager**, not in `.env` files that get committed. See
  [Secure configuration](../sdk/index.md#secure-configuration).

## Troubleshooting

**`401 Unauthorized: Invalid API key`**

Check that the key hasn't been revoked or expired (`GET /api/v1/api-keys` shows
`revoked_at` and `expires_at`), that you copied the whole key, and that your environment
variable is actually set in the process making the request.

**`403 Forbidden` on a request that should work**

The key's scopes don't cover that operation. Issue a new key with the scopes you need.

**`403` when creating a key**

Key creation is admin-only. Ask an organization admin to issue it.

## Related

- [Two-Factor Authentication](two-factor-authentication.md)
- [MCP Server](../integrations/mcp.md) — uses an API key via `WHITEBOXXAI_MCP_API_KEY`.
- [SDK Authentication](../sdk/api-reference.md#authentication)
