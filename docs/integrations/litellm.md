---
description: Record every LLM call that goes through your LiteLLM gateway in WhiteBoxXAI — connect it once, paste one block into LiteLLM's configuration, and see calls filed under the AI system they belong to, with the prompt and response redacted or left out as you choose.
---

# LiteLLM gateway

If your teams call models through a LiteLLM proxy, that proxy already sees every request, but nothing
records those requests as governance evidence. Each team would have to add WhiteBoxXAI logging to its own
code, and the ones that don't are the ones you can't see.

Connecting the gateway moves the recording to the one place every call passes through. An organisation
administrator connects it once in **Settings → LLM gateway**, adds one block to LiteLLM's configuration, and
from then on each call is stored in WhiteBoxXAI and filed under the
[AI system](../user-guide/model-register.md) it belongs to. Nobody changes their own code.

This page is written against LiteLLM 1.106.6. It records what the gateway sends; it does not check that the
gateway sends everything. See [What it cannot see](#what-it-cannot-see).

## How the data gets here

LiteLLM has a built-in callback, `generic_api`, that posts batches of its standard logging records to any
HTTPS endpoint. WhiteBoxXAI is that endpoint. LiteLLM pushes; WhiteBoxXAI does not poll your gateway, and
never holds your LiteLLM master key.

```text
LiteLLM proxy ── POST /api/v1/llm/gateways/litellm/ingest ──► WhiteBoxXAI
   (batch of up to 512 calls,                                  maps each call, applies your
    sent every 5 seconds)                                      content policy, files it
```

The gateway authenticates with its own token, which starts `wbx_lgw_`. It is not an API key: an API key acts
with the permissions of the administrator who made it, and a key pasted into a gateway configuration file
would carry all of them. The gateway token can do one thing, which is send calls for your organisation.

## Set it up

You need to be an administrator of your organisation. The gateway works on every plan; its calls count as
[Governance Events](../get-started/governance-events.md#litellm-gateway-calls).

1. **Connect.** In **Settings → LLM gateway**, on the **LiteLLM** tab, name the gateway and choose what is
   stored from each call (see [What is stored](#what-is-stored)), then select **Connect LiteLLM**.
   WhiteBoxXAI shows a token **once**. Copy it now; if you lose it, rotate it.
2. **Paste the configuration** into LiteLLM's `config.yaml`. The pane shows this block with your
   deployment's address filled in:

    ```yaml
    litellm_settings:
      callbacks: ["whiteboxxai"]
      # Requests sent with the no-log flag would otherwise never reach WhiteBox.
      global_disable_no_log_param: true

    callback_settings:
      whiteboxxai:
        callback_type: generic_api
        endpoint: os.environ/WHITEBOXXAI_INGEST_URL
        headers:
          Authorization: os.environ/WHITEBOXXAI_AUTHORIZATION
        event_types: ["llm_api_success", "llm_api_failure"]
        log_format: json_array
        max_retries: 3
        retry_delay: 2
        timeout: 30
    ```

3. **Set two environment variables** where LiteLLM runs:

    | Variable | Value |
    | --- | --- |
    | `WHITEBOXXAI_INGEST_URL` | `https://api.whiteboxxai.com/api/v1/llm/gateways/litellm/ingest` (shown in the pane; a Business Cloud tenant has its own address) |
    | `WHITEBOXXAI_AUTHORIZATION` | `Bearer wbx_lgw_...`, the whole header value including the word `Bearer` |

4. **Restart LiteLLM**, send a request through it, and wait about ten seconds. The pane changes from
   "Waiting for the first call" to "Receiving".

An organisation has one LiteLLM connection. If you run several LiteLLM proxies, give them all the same
configuration and token; their calls arrive as one gateway.

The pane also has tabs for OpenTelemetry (OTLP), Portkey AI, Kong AI Gateway and Azure APIM. This page
covers LiteLLM only; those tabs are not documented yet.

### Three details that matter

- **Use `callbacks`, not `success_callback`.** With `success_callback` or `failure_callback`, LiteLLM builds
  the logger and prints that it initialised it, but never calls it: nothing is sent and neither side
  reports an error. `event_types` still limits the callback to the two events listed.
- **Keep the retry settings.** LiteLLM's default is no retries, and a batch that fails once is thrown away.
  With `max_retries` set, LiteLLM retries timeouts, connection errors and server errors. It does not retry
  a refusal (see [When calls are lost](#when-calls-are-lost)).
- **`global_disable_no_log_param: true`** makes LiteLLM ignore the per-request switches that turn logging
  off, including the `x-litellm-disable-callbacks` header. Without it, a request sent with `no-log` never
  reaches WhiteBoxXAI.

## What is stored

You choose, per gateway, and can change it later. A change applies to calls received from then on; calls
already recorded keep the policy they were recorded under, and each row says which one that was.

| Policy | Stored |
| --- | --- |
| **Redact personal data (recommended)**, the default | Prompts, responses, messages and tool calls, with the patterns below removed. |
| **Metadata only** | No prompt, response, message, tool definition or tool call. You keep who called which model, tokens, cost, timing and the class of any error. |
| **Store everything as sent** | Exactly what the gateway sent, personal data included. |

### What redaction removes, and what it does not

Redaction matches patterns. It does not understand the text, so it has limits, and the pane states them
next to the choice.

It removes: email addresses, US-format phone numbers, 16-digit card numbers, US Social Security numbers,
IPv4 addresses, API keys, access tokens and private keys, and embedded images and files.

It does **not** find: names, postal addresses, phone numbers outside the US, American Express and other card
numbers that are not 16 digits, IBANs, and national ID numbers. If those must never be held, choose
**Metadata only**.

It walks every string in the call, including chat messages made of typed blocks and the arguments of tool
calls. Each stored row records which policy applied, the method (`pattern_v1`), and the kinds of thing it
removed.

## What is recorded from each call

| LiteLLM sends | Stored as |
| --- | --- |
| `custom_llm_provider` | Provider |
| `model` | Model name, as LiteLLM names it (for example `openai/gpt-4o`) |
| `messages`, `response` | Messages, prompt (the last user turn) and completion, under your policy |
| `prompt_tokens`, `completion_tokens`, `total_tokens` | Token counts |
| `response_cost` | Cost, **as LiteLLM reports it**. WhiteBoxXAI does not re-estimate it. If LiteLLM sends none, the cost is recorded as unavailable, not as zero. |
| `startTime`, `endTime` | Time of the call and latency |
| `status`, `error_information` | Outcome: success, failure, timeout or rate-limited, with the error |
| `litellm_call_id` | Request id, used to recognise a retried batch |
| `request_tags`, `session_id`, `end_user` | Tags (including any LiteLLM adds itself, such as `User-Agent`), session, and the end user |
| `model_parameters` | Temperature, maximum tokens, top-p, penalties, and tool definitions |
| key alias, team, team id, model group | The labels used to file the call under an AI system |

Never stored: the hash of the LiteLLM key, the key owner's email, the requester's IP address, custom request
headers, request metadata the caller attached, LiteLLM's spend-log metadata (`spend_logs_metadata`), and
LiteLLM's internal parameters. The address of the model
endpoint is kept without any credentials or query string.

## Filing calls under AI systems

A call is stored the moment it arrives. Which AI system it belongs to is decided by **rules**, evaluated
lowest priority number first, then oldest first:

- A rule maps a label to an AI system. The labels are the **team**, **team ID**, **API key** and **model**
  that LiteLLM reports, and **tags**.
- A **default** AI system, if you set one, takes every call that matches no rule. Use it only if one system
  really owns everything else, because it hides the traffic nobody has claimed.
- A call that matches nothing is still stored, with no AI system.

**Team and key labels exist only if you use LiteLLM's virtual keys and teams.** Without them, the only label
a call has is its model, and that is all the pane can list.

### Unregistered traffic

The pane lists, for the last 30 days, each team, key, model and tag label that no rule claims, with its
call count and when it was last seen. For a risk officer this is the answer to "what is flowing through our
gateway that nobody registered". A call can appear under several labels at once, so the per-label counts
add up to more than the number of calls; the heading shows the real figure as "(N distinct calls)".

If your calls carry no team or API key, the pane says so: only the model is known, and an assignment
applies to everything sent to that model. If you have set a default AI system, the pane shows how many
calls it has absorbed in the last 30 days, so the traffic it hides is still counted.

To claim a label, choose an AI system next to it and select **Assign**. If the system isn't registered
yet, [register it](../user-guide/model-register.md) and come back. To write a rule for a label that isn't
listed, use **Add rule**: pick the field (Tag, Team, Team ID, API key or Model), enter the value, choose the
AI system.

Assigning also files the last 30 days of unassigned matching calls under that system. It runs in the
background. The pane checks back for a few seconds and then says how many earlier calls were assigned, or
that they are still being assigned. Each re-filed call is stamped with how and when it was attributed
(`retroactive`, the rule, who assigned it), so the record shows a link made after the fact. A call that
already has an AI system is not moved. Removing a rule affects calls received from then on; calls already
filed keep their AI system.

## Health

The pane leads with a verdict in words, then what to do, then the counts.

| State | Meaning | What to do |
| --- | --- | --- |
| **Waiting for the first call** | Nothing has arrived. | Check the configuration, the two variables, and that LiteLLM was restarted. |
| **Receiving** | Calls are arriving. | Nothing. |
| **Receiving, with problems** | Calls arrive, but some records were rejected as unreadable. The reason is shown. | Usually a LiteLLM setting that changes the record format; see [Troubleshooting](#troubleshooting). |
| **Calls are being lost** | Batches were refused before they were recorded. The count of calls not recorded is shown. | See [When calls are lost](#when-calls-are-lost). |
| **Gone quiet** | It was receiving, and nothing has arrived for 24 hours. | Check the gateway is running and its configuration is unchanged. |
| **Disabled** | You switched acceptance off. | Switch it back on. |

The counts are totals since you connected: calls recorded, records rejected, and calls lost before they
arrived. The same verdict appears on the **Integrations** card, so a gateway that has stopped is visible
without opening the pane.

### When calls are lost

Two things refuse a batch before it is recorded: a Free plan that has used its monthly
[Governance Events](../get-started/governance-events.md), and the gateway's rate limit of 1,200 requests per
60 seconds. Both answer `429`. **LiteLLM does not resend a refused batch**, so those calls are not in your
evidence record. WhiteBoxXAI cannot recover them, but it does count them and says why, which is the part
that would otherwise be silent.

A request that is too large (`413`, see [Limits](#limits)) is also counted as lost, but it does not change
the verdict to **Calls are being lost**; only quota and rate-limit refusals do.

On Business Cloud the monthly allowance is not a hard cap, so only the rate limit applies.

## What it cannot see

WhiteBoxXAI records what the gateway sends. It cannot check that against anything.

- A call that does not go through a connected gateway is invisible: a team calling a provider directly, or a
  second gateway that isn't connected.
- A gateway whose configuration changed after setup, or that is stopped, sends nothing. A stopped gateway
  shows as **Gone quiet** after 24 hours, not before.
- There is no reconciliation against LiteLLM's own spend logs. It would catch a gap in either direction, and
  is not built yet.

So the pane says "Every call received in the last 30 days belongs to an AI system you have registered" only
when calls have arrived and none is unassigned. It never says every call is recorded.

## Governance Events

Each call that is **stored** is one Governance Event, the same as a call logged by the SDK. A call that is a
duplicate (LiteLLM re-sending a batch) or that WhiteBoxXAI rejected as unreadable costs nothing. See
[Governance Events](../get-started/governance-events.md#litellm-gateway-calls).

## Limits

| | |
| --- | --- |
| Records per request | 1,000. LiteLLM sends up to 512 by default, so you only reach this if you raise `DEFAULT_BATCH_SIZE`. A larger request is refused with `413` and counted as lost. |
| Request size | 32 MB. A larger body is refused with `413` and counted as lost. |
| Request rate | 1,200 per 60 seconds per gateway, separate from your dashboard and SDK traffic |
| Retention | Recorded calls follow your organisation's data retention policy, the same as calls logged with the SDK: 90 days on Free, 365 days by default on paid plans, which can be set between 1 and 7 years. Calls older than 30 days are compressed in storage. An administrator can remove older calls sooner with `DELETE /api/v1/llm/logs/cleanup?days=N` (`days` from 1 to 365), which only touches their own organisation. |

## Manage the connection

- **Rotate the token** replaces it. The old one stops working immediately, so the gateway is refused, and its
  calls lost, until you put the new token in `WHITEBOXXAI_AUTHORIZATION`. Rotating needs an administrator on
  a signed-in session, not an API key.
- **Accept calls from this gateway** switched off makes WhiteBoxXAI refuse everything it sends.
- **Disconnect** removes the LiteLLM connection and its rules. Calls already recorded are kept, and a
  gateway connected on another tab is not affected.

Every change, including connecting, rotating, each rule and each re-filing of earlier calls, is recorded in
the [audit trail](../user-guide/compliance-evidence.md). The token is never written to it.

## Troubleshooting

| You see | Cause |
| --- | --- |
| Still "Waiting for the first call" | The configuration uses `success_callback` instead of `callbacks`; LiteLLM wasn't restarted; or the variables aren't set where LiteLLM runs. LiteLLM's debug log (`--detailed_debug`) shows `Generic API Logger - sent batch ... status: 200` when it works. |
| `401` in LiteLLM's log | The token is wrong, was rotated, or the connection is disabled. WhiteBoxXAI answers all three the same way. |
| "Receiving, with problems" and `startTime/endTime are not numbers` | LiteLLM's `generic_api_use_v1` is on, which sends an older record shape. Turn it off. |
| `413` | A request carried more than 1,000 records. Lower `DEFAULT_BATCH_SIZE`. |
| `429` | A quota or rate-limit refusal; see [When calls are lost](#when-calls-are-lost). |
| Calls arrive with no AI system | No rule matches. Assign the label under **Unregistered traffic**. |

## Without the pane

The endpoint accepts LiteLLM's default `json_array` body: a JSON array of standard logging records, with
`Authorization: Bearer wbx_lgw_...`. It returns how many were stored:

```json
{ "accepted": 12, "duplicates": 0, "rejected": 0 }
```

A record it cannot read is counted in `rejected` and the rest of the batch is still stored, because an error
status would make LiteLLM resend the whole batch. A single record object is accepted too, for LiteLLM's
`single` log format. The administrator endpoints are under `/api/v1/integrations/llm-gateways/litellm`;
see the [API reference](../sdk/api-reference.md#litellm-gateway).
