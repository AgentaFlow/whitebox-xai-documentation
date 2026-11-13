---
description: Record LLM calls from OpenTelemetry, Portkey, Kong AI Gateway, Azure API Management and Microsoft Foundry in WhiteBoxXAI — what each gateway sends, the configuration to add, how calls are filed under AI systems, and the limits that apply to every gateway connection.
---

# LLM gateways

Your teams' LLM calls already pass through a gateway, or through code that emits OpenTelemetry traces.
Connecting it to WhiteBoxXAI records each call where it passes through, so no team has to add logging to its
own code. WhiteBoxXAI accepts calls from six sources:

| Gateway | What it sends | Where it goes |
| --- | --- | --- |
| [LiteLLM](litellm.md) | LiteLLM's standard logging records, in batches | `/api/v1/llm/gateways/litellm/ingest` |
| [OpenTelemetry (OTLP)](#opentelemetry-otlp) | OTLP traces over HTTP, JSON or Protobuf, from a Collector or an SDK | `/api/v1/llm/gateways/otlp/v1/traces` |
| [Portkey](#portkey) | Portkey's OpenTelemetry log export | `/api/v1/llm/gateways/otlp/v1/traces` |
| [Kong AI Gateway](#kong-ai-gateway) | The `http-log` plugin's record, with the `ai-proxy` fields | `/api/v1/llm/gateways/kong/ingest` |
| [Azure API Management](#azure-api-management) | A JSON record from a `send-one-way-request` policy | `/api/v1/llm/gateways/azure_apim/ingest` |
| [Microsoft Foundry](microsoft-foundry.md) | OpenTelemetry traces from the Foundry client libraries, or forwarded Azure Monitor logs | `/api/v1/llm/gateways/otlp/v1/traces`, `/api/v1/llm/gateways/microsoft_foundry/ingest` |

All addresses are on `https://api.whiteboxxai.com`; a Business Cloud tenant has its own address, which the
setup pane shows.

LiteLLM has its own guide: [LiteLLM gateway](litellm.md). [Microsoft Foundry](microsoft-foundry.md) has its own guide too. This page covers the other four, and what all six
connections share.

## What every connection shares

### Connect

You need to be an administrator of your organisation.

1. In **Settings → LLM gateway**, open the gateway's tab: **OpenTelemetry (OTLP)**, **Portkey AI**,
   **Kong AI Gateway** or **Azure APIM**.
2. The tab says what that gateway sends. Name the connection, choose what is stored from each call, and
   select **Connect**. Azure APIM sends no prompt or response, so its tab shows a note in place of that
   choice. WhiteBoxXAI shows a token starting `wbx_lgw_` **once**, as `Bearer wbx_lgw_...`. Copy it before
   you switch tab; switching tab clears it. If you lose it, rotate it.
3. Put the token and the address into the gateway's configuration, as each section below shows. The token
   banner says where this gateway takes the token, because it is not the same place on every gateway:

    | Gateway | Where the token goes |
    | --- | --- |
    | OpenTelemetry, through a Collector | `WHITEBOXXAI_AUTHORIZATION`, where the Collector runs |
    | OpenTelemetry, from an SDK | Inside `OTEL_EXPORTER_OTLP_TRACES_HEADERS`, as `Authorization=Bearer%20wbx_lgw_...` |
    | Portkey | Inside `EXPERIMENTAL_GEN_AI_OTEL_EXPORTER_OTLP_HEADERS` on your Portkey gateway, as `Authorization=Bearer wbx_lgw_...` |
    | Kong AI Gateway | `DECK_WHITEBOXXAI_AUTHORIZATION`, where you run decK |
    | Azure APIM | A secret named value called `WHITEBOXXAI_AUTHORIZATION` |
    | Microsoft Foundry | Inside `OTEL_EXPORTER_OTLP_TRACES_HEADERS` where your app runs, as `Authorization=Bearer%20wbx_lgw_...`; a forwarder takes it as `WHITEBOXXAI_AUTHORIZATION` |

4. Send a call through the gateway. The tab changes from **Waiting for the first call** to **Receiving**.

An organisation has one connection per gateway type, so up to six at once. Each has its own token, rules,
health and rate limit. A token only works for its own gateway type: a Kong token sent to the OTLP address is
refused with `401`. The exceptions are Portkey and Microsoft Foundry, whose tokens are accepted on the OTLP address because that
is where they send.

### What is stored, and how calls are filed

These work the same way for every gateway, and are described once in the LiteLLM guide:

- **Content policy**: **Redact personal data (recommended)**, **Metadata only** or **Store everything as
  sent**, applied to every prompt, message and response a gateway sends. See
  [What is stored](litellm.md#what-is-stored) and
  [what redaction removes, and what it does not](litellm.md#what-redaction-removes-and-what-it-does-not).
- **Rules and unregistered traffic**: rules map a label to an AI system, unclaimed labels from the last 30
  days are listed so you can assign them, and assigning also files the last 30 days of matching calls. See
  [Filing calls under AI systems](litellm.md#filing-calls-under-ai-systems).
- **Health**: the same six states, from **Waiting for the first call** to **Calls are being lost**. See
  [Health](litellm.md#health).

The labels are always called **Team**, **Team ID**, **API key**, **Model** and **Tag**, but each gateway fills
them from different fields. On Kong, for example, **API key** holds the consumer's username. Each section below
has a table showing where each label comes from.

If a call carries no team or key, only its model is known, and the pane says so. An assignment then applies
to every call to that model.

### Duplicates

A call is identified by its request id (each section below says which field). A call that arrives twice is
stored once and costs nothing the second time. This is per organisation: the same request id arriving
through two gateways is also stored once.

### Limits

| | |
| --- | --- |
| Records per request | 1,000. For OpenTelemetry and Portkey, this counts GenAI spans only. A larger request is refused with `413` and counted as lost. |
| Request size | 32 MB, read from the `Content-Length` header. A larger request is refused with `413` and counted as lost. |
| Request rate | 1,200 per 60 seconds for each connection, separate from your dashboard and SDK traffic. Beyond that, `429`. |
| Free plan | A request that would record more Governance Events than are left this month is refused whole with `429`. On Business Cloud the monthly allowance is not a hard cap. |
| Retention | As for [LiteLLM calls](litellm.md#limits): your organisation's data retention policy. |

A refused request is counted under **Calls are being lost**. Whether the gateway sends it again depends on
the gateway. LiteLLM and Portkey do not, and the pane says so. For OpenTelemetry, Kong and Azure API
Management, their retry behaviour has not been confirmed, so the pane tells you to check whether your
gateway sends the calls again. Until it does, treat the count as calls missing from your record.

### Governance Events

Each call **stored** is one [Governance Event](../get-started/governance-events.md#llm-gateway-calls). For
OpenTelemetry and Portkey, that is one event per GenAI span stored. Duplicates, rejected records, spans that
are not GenAI, and requests that cannot be read at all cost nothing.

### The Integrations card

**Settings → Integrations** has one card for all gateway connections, titled **LLM gateways**. Its count,
**N calls recorded**, is the total across every connection. It shows the state of the connection that most
needs attention, in this order: calls being lost, receiving with problems, gone quiet, receiving, waiting,
disabled. Where two are tied, LiteLLM is shown.

Unless that state is **Receiving**, the card shows the verdict, starting with the gateway's name, for
example "Kong AI Gateway: …". So a stopped Kong connection is visible there, and named, even if LiteLLM is
fine.

## OpenTelemetry (OTLP)

Use this for an OpenTelemetry Collector, or for an application whose OpenTelemetry SDK exports traces
directly. WhiteBoxXAI accepts OTLP over HTTP, as JSON or Protobuf, optionally gzip-compressed. It does not
accept OTLP over gRPC.

### Set it up through a Collector

Add an `otlphttp` exporter to your Collector's configuration and include it in your traces pipeline. The
pane shows this block:

```yaml
exporters:
  otlphttp/whiteboxxai:
    traces_endpoint: "${env:WHITEBOXXAI_OTLP_ENDPOINT}"
    headers:
      Authorization: "${env:WHITEBOXXAI_AUTHORIZATION}"

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch]
      exporters: [otlphttp/whiteboxxai]
```

The block assumes your Collector already defines the `otlp` receiver and `batch` processor; merge it into
your existing pipeline. Then set two environment variables where the Collector runs:

| Variable | Value |
| --- | --- |
| `WHITEBOXXAI_OTLP_ENDPOINT` | `https://api.whiteboxxai.com/api/v1/llm/gateways/otlp/v1/traces` |
| `WHITEBOXXAI_AUTHORIZATION` | `Bearer wbx_lgw_...`, including the word `Bearer` |

Use `traces_endpoint`, not `endpoint`. With `endpoint`, the exporter appends `/v1/traces` to an address that
already ends in it, and every request fails with `404`.

### Set it up from an SDK

To export straight from an application, set the standard OpenTelemetry variables:

| Variable | Value |
| --- | --- |
| `OTEL_EXPORTER_OTLP_TRACES_ENDPOINT` | `https://api.whiteboxxai.com/api/v1/llm/gateways/otlp/v1/traces` |
| `OTEL_EXPORTER_OTLP_TRACES_HEADERS` | `Authorization=Bearer%20wbx_lgw_...` |

The space after `Bearer` is written as `%20`, which is how the OpenTelemetry specification encodes it in
this variable.

### Which spans are kept

A trace export contains every span: HTTP requests, database calls, and the LLM calls you want. WhiteBoxXAI
keeps a span only if it has an attribute starting `gen_ai.`, `llm.` or `openinference.`, or if its name
contains `chat`, `llm`, `completion`, `generate`, `openai` or `anthropic`. Every other span is dropped
without being counted, billed or reported.

So an exporter that sends only non-GenAI spans leaves the tab at **Waiting for the first call**. If that
happens, check that your LLM client library is instrumented with OpenTelemetry's GenAI conventions.

### What is recorded from each span

Attributes are read from the span, then from its resource. Where several are listed, the first one present
wins.

| Stored as | From |
| --- | --- |
| Model | `gen_ai.response.model`, `gen_ai.request.model`, `llm.model_name` |
| Provider | `gen_ai.system`, `gen_ai.provider.name`, `llm.provider` |
| Messages and prompt | `gen_ai.system_instructions` and `gen_ai.input.messages`; otherwise prompt events, `gen_ai.prompt`, `llm.prompt` or `input.value` |
| Completion | `gen_ai.output.messages`; otherwise completion events, `gen_ai.completion`, `llm.completion` or `output.value` |
| Tokens | `gen_ai.usage.input_tokens` and `gen_ai.usage.output_tokens` (or the older `prompt_tokens` and `completion_tokens` names); the total if sent, otherwise their sum |
| Cost | `gen_ai.usage.cost` or `gen_ai.usage.total_cost`, as sent. If no cost is sent, the cost is recorded as unavailable, not as zero. |
| Time and latency | The span's start and end time |
| Outcome | Error if the span's status is error. A message mentioning a timeout is recorded as a timeout, and one mentioning a rate limit as rate-limited. |
| Request id | `gen_ai.response.id`, otherwise the span id |
| Session | `gen_ai.conversation.id` or `session.id`, otherwise the trace id |
| End user | `enduser.id` or `user.id` |
| Environment | The resource's `deployment.environment` |
| Parameters | `gen_ai.request.temperature`, `max_tokens`, `top_p`, `frequency_penalty`, `presence_penalty` |

### Labels for filing

| Label | From |
| --- | --- |
| Team | `gen_ai.team` or `team`, otherwise the resource's `service.name` |
| Team ID | `gen_ai.team_id`, or the resource's `team.id` |
| API key | `client.id`, otherwise `enduser.id` |
| Model | `gen_ai.request.model`, otherwise the model |
| Tag | `gen_ai.tags` or `tags` (a list or a comma-separated string), and the resource's `service.namespace` |

Because the team falls back to `service.name`, each instrumented service usually shows up as its own team
under **Unregistered traffic**. That is often the most useful way to assign calls to AI systems.

## Portkey

Portkey sends calls to WhiteBoxXAI through its OpenTelemetry log export, which Portkey calls Complete Logs
Export. Portkey describes it as experimental and offers it on Enterprise and self-hosted gateways only. It
sends each call on its own, without batching, and does not retry a failed send.

### Set it up

Connect on the **Portkey AI** tab, then set these environment variables on your Portkey gateway:

```yaml
EXPERIMENTAL_GEN_AI_OTEL_TRACES_ENABLED: "true"
EXPERIMENTAL_GEN_AI_OTEL_EXPORTER_OTLP_ENDPOINT: "https://api.whiteboxxai.com/api/v1/llm/gateways/otlp"
EXPERIMENTAL_GEN_AI_OTEL_EXPORTER_OTLP_HEADERS: "Authorization=Bearer wbx_lgw_..."
```

The endpoint stops at `/otlp`, without `/v1/traces`, because Portkey adds `/v1/traces` itself. The header is
written with a plain space, as Portkey's documentation shows.

### What Portkey records

Portkey's spans are read exactly as described under
[What is recorded from each span](#what-is-recorded-from-each-span), with three additions:

- **Cost** comes from `portkey.usage.cost` first.
- **Team** comes from `portkey.metadata.team`, otherwise from `portkey.workspace.slug`.
- **Other metadata** you attach in Portkey becomes a tag such as `region:eu`. Metadata named `team_id`,
  `key_alias` or `model_group` fills that label instead. The workspace is also added as a tag,
  `workspace:<slug>`.

To file Portkey calls by team, attach `team` metadata to requests in Portkey. Without it, calls are grouped by
workspace, and without a workspace, only by model.

## Kong AI Gateway

Kong's `ai-proxy` plugin records each LLM call in Kong's log, and its `http-log` plugin sends that log to
WhiteBoxXAI.

### Set it up

1. **Turn on AI logging.** In your existing `ai-proxy` or `ai-proxy-advanced` plugin, set:

    ```yaml
    config:
      logging:
        log_statistics: true
        log_payloads: true
    ```

    Without `log_statistics` Kong leaves out token counts, and without `log_payloads` it leaves out the prompt
    and response. Add these settings to your existing plugin; don't declare a second `ai-proxy`, which would
    replace your model and route settings.

2. **Add `http-log`.** The pane shows this decK configuration:

    ```yaml
    _format_version: "3.0"
    plugins:
      - name: http-log
        config:
          http_endpoint: ${{ env "DECK_WHITEBOXXAI_INGEST_URL" }}
          method: POST
          content_type: application/json
          timeout: 10000
          keepalive: 60000
          flush_timeout: 2
          headers:
            Authorization: ${{ env "DECK_WHITEBOXXAI_AUTHORIZATION" }}
    ```

    As written, the plugin applies to every service. To limit it to your AI routes, add a `service` or
    `route` to the plugin entry.

3. **Set the variables where you run decK**, then run `deck gateway sync`:

    | Variable | Value |
    | --- | --- |
    | `DECK_WHITEBOXXAI_INGEST_URL` | `https://api.whiteboxxai.com/api/v1/llm/gateways/kong/ingest` |
    | `DECK_WHITEBOXXAI_AUTHORIZATION` | `Bearer wbx_lgw_...` |

    decK only substitutes variables whose names start with `DECK_`, and it substitutes them when it syncs.
    The token is then held in Kong's plugin configuration.

`http-log` can send one record per request or, with its queue's batch size above 1, an array of records.
WhiteBoxXAI accepts both.

### What Kong records

| Stored as | From |
| --- | --- |
| Model | The `ai-proxy` request or response model, otherwise the model in the request or response body |
| Provider | The `ai-proxy` provider name, otherwise `kong-ai` |
| Tokens and cost | The `ai-proxy` usage figures, otherwise the response body's `usage`, otherwise Kong's `x-llm-usage-*` headers |
| Messages, prompt and completion | The `ai-proxy` payloads, otherwise the request and response bodies |
| Time and latency | `started_at`, and Kong's request latency |
| Outcome | The response status: `429` is rate-limited, `408` and `504` are timeouts, other `4xx` and `5xx` are failures |
| Request id | Kong's request id |
| End user | The consumer's username |

A record without `started_at` is rejected.

### Labels for filing

| Label | From |
| --- | --- |
| Team | The consumer's `custom_id`, otherwise the service name |
| Team ID | The consumer's id |
| API key | The consumer's username |
| Model | The model |
| Tag | The route's tags |

To file Kong calls by team, give each team's Kong consumer a `custom_id`.

## Azure API Management

An outbound policy on your Azure OpenAI API sends one JSON record per call to WhiteBoxXAI. It records the
model, token counts, timing and outcome. It does not send the prompt, the response or a cost, so the content
policy has nothing to redact, and the **Azure APIM** tab shows a note instead of the content-policy choice.

### Set it up

1. **Create two named values** in your API Management instance:

    | Named value | Value |
    | --- | --- |
    | `WHITEBOXXAI_INGEST_URL` | `https://api.whiteboxxai.com/api/v1/llm/gateways/azure_apim/ingest` |
    | `WHITEBOXXAI_AUTHORIZATION` | `Bearer wbx_lgw_...`. Make this a secret named value. |

2. **Add the policy** inside your API's `<outbound>` section, after `<base />`. The pane shows this snippet:

    ```xml
    <send-one-way-request mode="new">
        <set-url>{{WHITEBOXXAI_INGEST_URL}}</set-url>
        <set-method>POST</set-method>
        <set-header name="Authorization" exists-action="override">
            <value>{{WHITEBOXXAI_AUTHORIZATION}}</value>
        </set-header>
        <set-header name="Content-Type" exists-action="override">
            <value>application/json</value>
        </set-header>
        <set-body>@{
            var deployment = context.Request.MatchedParameters.GetValueOrDefault("deployment-id",
                (string)context.Variables.GetValueOrDefault("deployment-id", ""));
            JObject respObj = null;
            var contentType = context.Response.Headers.GetValueOrDefault("Content-Type", "");
            if (contentType.Contains("application/json")) {
                try {
                    respObj = context.Response.Body?.As&lt;JObject&gt;(preserveContent: true);
                } catch {}
            }
            return new JObject(
                new JProperty("request_id", context.RequestId.ToString()),
                new JProperty("timestamp", context.Timestamp.ToString("o")),
                new JProperty("deployment", deployment),
                new JProperty("model", (string)respObj?["model"] ?? deployment),
                new JProperty("usage", respObj?["usage"]),
                new JProperty("api_id", context.Api?.Id),
                new JProperty("durationMs", context.Elapsed.TotalMilliseconds),
                new JProperty("subscription_id", context.Subscription?.Id),
                new JProperty("product_id", context.Product?.Id),
                new JProperty("status_code", context.Response.StatusCode)
            ).ToString();
        }</set-body>
    </send-one-way-request>
    ```

    `send-one-way-request` does not wait for WhiteBoxXAI to answer, so it adds no delay to your callers.

### What Azure API Management records

| Stored as | From |
| --- | --- |
| Model | The **deployment name** from the request URL. If the API has no `deployment-id` parameter, the model the response reports. |
| Provider | `azure_openai` |
| Tokens | The response's `usage`, for JSON responses only |
| Cost | Not sent, so recorded as unavailable |
| Time and latency | The request's start time and APIM's elapsed time |
| Outcome | The response status: `429` is rate-limited, `408` and `504` are timeouts, other `4xx` and `5xx` are failures |
| Request id | APIM's request id |

**Streamed responses record no tokens.** The policy reads the response body only when it is JSON, because
reading a streamed (server-sent events) response would make APIM buffer it and break streaming for your
callers. A streamed call is still recorded, with its model, timing and outcome.

### Labels for filing

| Label | From |
| --- | --- |
| Team | The APIM product |
| API key | The APIM subscription |
| Model | The APIM API's id |

The **Model** label holds the API's id, not the model name, so **Unregistered traffic** lists one row per
API. To file calls by team, put each team's subscriptions under its own product.

## Troubleshooting

| You see | Cause |
| --- | --- |
| `401` in the gateway's log | The token is wrong, was rotated, or the connection is disabled. Or it belongs to another gateway's tab: each gateway type needs its own connection and token. |
| `404` from the OpenTelemetry Collector | The exporter uses `endpoint` instead of `traces_endpoint`, so it sends to `.../v1/traces/v1/traces`. |
| `404` from Portkey | The endpoint ends in `/v1/traces`. Stop it at `/otlp`; Portkey adds the rest. |
| Traces are sent, but the tab stays at **Waiting for the first call** | None of the spans is a GenAI span. See [Which spans are kept](#which-spans-are-kept). |
| **Receiving, with problems** and "OTLP payload is neither valid JSON nor valid Protobuf trace data" | The body could not be read as an OTLP trace export, for example because it was compressed with something other than gzip. The request costs nothing. |
| Kong calls with no tokens, or no prompt and response | `log_statistics` or `log_payloads` is not on in `ai-proxy`. |
| Kong records rejected with "started_at is not a number" | The record was not produced by `http-log`. |
| Azure calls with zero tokens | The calls are streamed. See [What is recorded](#what-azure-api-management-records). |
| `413` | A request was over 32 MB, or carried more than 1,000 records or GenAI spans. Lower the gateway's batch size. |
| `429` | A quota or rate-limit refusal. See [When calls are lost](litellm.md#when-calls-are-lost). |
| Calls arrive with no AI system | No rule matches. Assign the label under **Unregistered traffic**. |
