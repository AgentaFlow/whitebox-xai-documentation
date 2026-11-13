---
description: Record the calls your Microsoft Foundry projects and agents make in WhiteBoxXAI. Three ways in (OpenTelemetry tracing, forwarded Azure Monitor logs, and a Python SDK wrapper), what each one carries and does not, how calls are filed by Foundry project and agent, and what is shown of Azure AI Content Safety.
---

# Microsoft Foundry

Calls to models and agents in Microsoft Foundry (formerly Azure AI Foundry) happen in your applications and
in Foundry itself. Connecting Foundry to WhiteBoxXAI records each call where it passes through, so no team has
to add logging to its own code, and files it under the AI system it belongs to.

There are three ways in. They carry different things, so pick by what you need to see:

| Way in | What it carries | Where it goes |
| --- | --- | --- |
| [OpenTelemetry tracing](#set-it-up-through-tracing) from `azure-ai-projects` or `azure-ai-inference` | Model, tokens, timing, agent, thread and run; the prompt and response if you turn content recording on | `/api/v1/llm/gateways/otlp/v1/traces`, with a Microsoft Foundry token |
| [Azure Monitor diagnostic logs](#forward-azure-monitor-logs), forwarded from an Event Hub | That a call happened: time, deployment, duration, outcome. No prompt, response, tokens or cost | `/api/v1/llm/gateways/microsoft_foundry/ingest` |
| [The Python SDK wrapper](#wrap-a-client-in-python) | Model, tokens, timing, prompt and response, and Azure AI Content Safety's per-category results | `/api/v1/llm/logs`, with your API key |

All addresses are on `https://api.whiteboxxai.com`; a Business Cloud tenant has its own address, which the
setup pane shows.

The first two use a gateway connection and share its behaviour. This page covers what is specific to Foundry;
the rest is in [LLM gateways](gateways.md) and the [LiteLLM guide](litellm.md). The SDK wrapper does not use a
connection at all.

## Connect

You need to be an administrator of your organisation.

1. In **Settings → LLM gateway**, open the **Microsoft Foundry** tab. It is the first tab, and the one the
   pane opens on.
2. Name the connection, choose what is stored from each call, and select **Connect**. WhiteBoxXAI shows a
   token starting `wbx_lgw_` **once**, as `Bearer wbx_lgw_...`. Copy it before you switch tab; switching tab
   clears it. If you lose it, rotate it.
3. Put the token and the address where your app or forwarder runs, as the sections below show.
4. Send a call. The tab changes from **Waiting for the first call** to **Receiving**.

An organisation has one connection per gateway type, and a token only works for its own type. The content
policy, rules, health, limits and Governance Events work as described in
[LLM gateways](gateways.md#what-every-connection-shares).

## Set it up through tracing

Foundry's client libraries emit OpenTelemetry spans for chat calls and agent runs once you turn the
instrumentor on. Send them to WhiteBoxXAI with the standard OpenTelemetry exporter. The pane shows this block:

```python
# pip install azure-ai-projects opentelemetry-sdk opentelemetry-exporter-otlp-proto-http
from azure.ai.projects.telemetry import AIProjectInstrumentor
from opentelemetry import trace
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.resources import Resource
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor

provider = TracerProvider(
    resource=Resource.create({"foundry.project": "<your Foundry project name>"})
)
provider.add_span_processor(BatchSpanProcessor(OTLPSpanExporter()))
trace.set_tracer_provider(provider)

AIProjectInstrumentor().instrument()
```

Then set two environment variables where the app runs:

| Variable | Value |
| --- | --- |
| `OTEL_EXPORTER_OTLP_TRACES_ENDPOINT` | `https://api.whiteboxxai.com/api/v1/llm/gateways/otlp/v1/traces` |
| `OTEL_EXPORTER_OTLP_TRACES_HEADERS` | `Authorization=Bearer%20wbx_lgw_...` |

The space after `Bearer` is written as `%20`, which is how the OpenTelemetry specification encodes it in this
variable. WhiteBoxXAI accepts OTLP over HTTP, as JSON or Protobuf, and not over gRPC.

### Content recording is off until you turn it on

The Foundry libraries record **no prompt and no response** by default. Until you turn it on, calls arrive with
model, tokens, timing and outcome only, and the content policy you chose has nothing to apply to. The switch
is not the same in every library:

| Library | To record content |
| --- | --- |
| `azure-ai-projects` | `OTEL_INSTRUMENTATION_GENAI_CAPTURE_MESSAGE_CONTENT=true`, or `instrument(enable_content_recording=True)` |
| `azure-ai-inference` | `AZURE_TRACING_GEN_AI_CONTENT_RECORDING_ENABLED=true` |

When you turn it on, the content policy applies as for any gateway: personal data is redacted by pattern
unless you chose otherwise, and [what redaction removes, and what it does not](litellm.md#what-redaction-removes-and-what-it-does-not)
applies in full.

### The project name is yours to set

The Foundry instrumentor records which model, agent and endpoint a call used. It does **not** record the
Foundry project's name. WhiteBoxXAI reads the project from a resource attribute called `foundry.project`, as in
the block above. If you do not set it, the project falls back to the resource's `service.name` (unless that is
OpenTelemetry's default `unknown_service`, which is ignored), and without either, the call has no project label.

### Which spans are kept

As for [any OpenTelemetry export](gateways.md#which-spans-are-kept): a span is kept if it carries a
`gen_ai.` attribute. Foundry's chat and agent spans do. Other spans in the trace are dropped without being
counted, billed or reported.

### What is recorded from each span

| Stored as | From |
| --- | --- |
| Provider | Always `microsoft_foundry` for this connection. The library's own value (`microsoft.foundry`, `az.ai.inference`) is kept in the call's gateway details. |
| Model | `gen_ai.response.model`, otherwise `gen_ai.request.model` |
| Tokens | `gen_ai.usage.input_tokens` and `gen_ai.usage.output_tokens`; the total if sent, otherwise their sum |
| Agent | `gen_ai.agent.id`, `gen_ai.agent.name` and `gen_ai.agent.version`, in the call's gateway details |
| Thread and run | `gen_ai.thread.id` (or `gen_ai.conversation.id`) and `gen_ai.thread.run.id`, in the call's gateway details |
| Session | `gen_ai.conversation.id` or `session.id`, otherwise the trace id |
| Time and latency | The span's start and end time |
| Outcome | Error if the span's status is error |
| Request id | `gen_ai.response.id`, otherwise the span id |
| Cost | Recorded as unavailable. Foundry's tracing does not include it. |
| Prompt and response | Only if content recording is on (see above) |

## Forward Azure Monitor logs

Azure Monitor can record each request to a Foundry resource as a diagnostic log. It cannot send them to
WhiteBoxXAI directly: the destinations are a Log Analytics workspace, a storage account or an Event Hub. To
use these logs, stream them to an Event Hub and forward them from there.

**What these logs carry.** That a call happened: its time, deployment, duration and result. Microsoft's native
logging does not include the prompt, the response, token counts or cost, and WhiteBoxXAI does not invent them.
The pane shows these calls with **no tokens recorded**, not with zero tokens. If you need to see usage, use
tracing or the SDK wrapper instead, or as well.

**What is never stored.** The caller's IP address and identity are in the log record. WhiteBoxXAI does not
keep either.

### Steps

1. On the Foundry resource, open **Diagnostic settings** and add a setting that sends the
   **RequestResponse** category to **Stream to an event hub**.
2. Create a function that reads that Event Hub and posts the records to WhiteBoxXAI. The ingest address takes
   the Event Hub's own `{"records": [...]}` body as it is, so the function only has to pass the records on.
   This is a sketch, in the Azure Functions Python v2 model. WhiteBoxXAI has not run it against a live Event
   Hub, so test it before you rely on it:

    ```python
    import json
    import os

    import azure.functions as func
    import requests

    app = func.FunctionApp()


    @app.event_hub_message_trigger(
        arg_name="events",
        event_hub_name="%EVENT_HUB_NAME%",
        connection="EVENT_HUB_CONNECTION",
        cardinality="many",
    )
    def forward_to_whiteboxxai(events: list[func.EventHubEvent]) -> None:
        records = []
        for event in events:
            records.extend(json.loads(event.get_body().decode("utf-8")).get("records", []))
        for start in range(0, len(records), 1000):  # WhiteBoxXAI takes up to 1,000 per request
            response = requests.post(
                os.environ["WHITEBOXXAI_FORWARDER_URL"],
                json={"records": records[start : start + 1000]},
                headers={"Authorization": os.environ["WHITEBOXXAI_AUTHORIZATION"]},
                timeout=30,
            )
            response.raise_for_status()  # a failed batch is retried by the Event Hub trigger
    ```

3. Set `WHITEBOXXAI_FORWARDER_URL` to `https://api.whiteboxxai.com/api/v1/llm/gateways/microsoft_foundry/ingest`
   and `WHITEBOXXAI_AUTHORIZATION` to `Bearer wbx_lgw_...`.

The Event Hub, the function and the diagnostic setting run in your Azure subscription and are billed there.
A record that is sent twice is stored once, because the log's `correlationId` is its request id.

### What is recorded from each log record

| Stored as | From |
| --- | --- |
| Time | `time`. A record with no valid time is rejected, not stamped with the time it arrived. |
| Model | `properties.modelDeploymentName`, then `properties.modelName`. If neither is present the model is `unknown`. |
| Latency | `durationMs` |
| Outcome | `resultSignature` as an HTTP status: `429` is rate-limited, `408` and `504` are timeouts, anything of `400` or above is a failure |
| Request id | `correlationId` |
| Project and resource | Read from `resourceId`: the account name, and the project if the id names one. Lower-cased, so they match the labels a trace gets from the endpoint host. Set `foundry.project` in lower case to match the project too. |
| Tokens, cost, prompt, response | Not available |

The names inside a log's `properties` object vary by Azure resource and API version. WhiteBoxXAI reads the two
above and ignores the rest. If your records use another name for the deployment, the call is stored under
`unknown`; tell us and we will add it.

## Wrap a client in Python

The SDK wrapper records chat calls from inside your application. It needs no gateway connection: it logs with
your API key, to the AI system you name.

```python
from azure.ai.inference import ChatCompletionsClient
from whiteboxxai import WhiteBoxXAI
from whiteboxxai.integrations.foundry import FoundryMonitor, wrap_foundry_client

wbx = WhiteBoxXAI(api_key="your-api-key")
monitor = FoundryMonitor(wbx, model_id="<AI system id>", project_name="claims-bot")

chat = wrap_foundry_client(ChatCompletionsClient(endpoint, credential), monitor)
reply = chat.complete(messages=[...], model="gpt-4o")  # recorded as it returns
```

`wrap_foundry_client` returns a proxy that behaves like the client you gave it. It is not an instance of the
client's class, so an `isinstance` check on it fails. It does not import any Azure package, so it adds nothing
to your dependencies.

**What it wraps.** `complete()` on `ChatCompletionsClient`, and `chat.completions.create()` on `AzureOpenAI`
and `AsyncAzureOpenAI`, sync or async. Given an `AIProjectClient`, it wraps the clients that
`get_openai_client()` (and, on older releases, `inference.get_chat_completions_client()`) hands out.

**What it does not wrap.** The Responses API, embeddings, and Foundry Agent Service runs. Use tracing for
those: it sees agent runs and threads.

**What it records.** Model, tokens, latency, finish reason, the prompt and response, the project, hub and agent
you name (on the monitor, or per client with `project_name=`, `hub_name=` and `agent_id=`), and Azure AI
Content Safety's result when the response or error carries one.

| Option on `FoundryMonitor` | Default | Effect |
| --- | --- | --- |
| `capture_content` | `True` | `False` sends no prompt, response or messages: only model, tokens, timing, outcome and the Content Safety result. An error is reduced to its class name, because a service error often quotes the prompt. |
| `mask_content` | `True` | Masks personal data in what is stored, using the SDK's pattern list: email addresses, US-format phone numbers, US Social Security numbers, 16-digit card numbers and IPv4 addresses. Names, postal addresses, non-US phone numbers and other ID numbers are not caught. |

**Streaming.** A streamed call is recorded when the stream opens, with no tokens and no completion, and marked
as streamed. Its latency is the time until the stream opened, and the record says so.

**It never changes your call.** If recording fails, you get a warning in your log and your call returns as
normal. If Foundry returns an error, it is recorded and then raised again unchanged.

## Labels for filing

Calls are filed under AI systems by [rules](litellm.md#filing-calls-under-ai-systems) on these labels:

| Label | From a trace | From a log | From the SDK wrapper |
| --- | --- | --- | --- |
| Team | The `foundry.project` resource attribute, otherwise `service.name` | The project in `resourceId` | `project_name` |
| Team ID | `foundry.hub`, otherwise the resource named in the endpoint (`<name>.services.ai.azure.com`, `.openai.azure.com` or `.cognitiveservices.azure.com`) | The account in `resourceId` | `hub_name` |
| API key | `gen_ai.agent.id`, otherwise `gen_ai.agent.name` | None | `agent_id` |
| Model | The model | The deployment | The model |

For the SDK wrapper, the labels are kept on the record; the AI system is the one you passed as `model_id`, so
no rule is needed. For tracing and logs, a call with no matching rule is listed under **Unregistered
traffic** until you assign its label.

## Azure AI Content Safety

Azure OpenAI and Foundry model deployments apply Azure AI Content Safety guardrails, which you configure per
deployment in Foundry. When Azure reports a result for a call, WhiteBoxXAI stores it with the call. Once the
call is filed under an AI system, the conversation log on **Observability → LLM** shows it in words, for example:

> Azure AI Content Safety blocked this prompt: jailbreak attempt.

This is **Microsoft's verdict**. WhiteBoxXAI copies it from what the service returned. It does not run its own
check, and it does not change the result.

- A **block** and a **flag** are different. A jailbreak that Azure detected but did not block is shown as
  flagged, and the line says it was not blocked. Low-severity results that Azure did not filter are not shown
  as findings.
- A call with no verdict shows nothing. That is not the same as passing: it means Azure sent no result.
- A call that Azure examined and flagged nothing on also shows nothing.

**Which way in carries what.** The per-category results (`prompt_filter_results` and `content_filter_results`
on a chat response, `content_filters` on a Responses API response) and a refused prompt (HTTP `400` with the
error code `content_filter`) are in the response or error body. Only the SDK wrapper, or a forwarder you write
that posts the response body, sees them. The Foundry tracing spans do not carry those results: from tracing, a
response that Azure cut off (`finish_reason` of `content_filter`) is marked as blocked, and that is all.
Azure Monitor logs carry none.

## Troubleshooting

| You see | Cause |
| --- | --- |
| `401` from the exporter or the function | The token is wrong, was rotated, or the connection is disabled. Or it belongs to another gateway's tab: a Microsoft Foundry token works on the OTLP traces address and the `microsoft_foundry` ingest address, and no other. |
| The tab stays at **Waiting for the first call** | No span with a `gen_ai.` attribute has arrived. Check that `AIProjectInstrumentor().instrument()` runs before the first call, and that the exporter's endpoint is the traces address. |
| Calls arrive with no prompt or response | Content recording is off in the app. See [Content recording is off until you turn it on](#content-recording-is-off-until-you-turn-it-on). |
| Calls from logs show no tokens | Azure Monitor logs do not carry them. This is expected. |
| Calls from logs show the model as `unknown` | The log's `properties` has no `modelDeploymentName` or `modelName`. |
| Calls arrive with no project | The `foundry.project` resource attribute is not set. See [The project name is yours to set](#the-project-name-is-yours-to-set). |
| **Receiving, with problems** and "Microsoft Foundry record has no valid time or timestamp" | A forwarded record has no `time` field, or one that is not a date. |
| No Content Safety line on a call | Azure sent no result with it, or the call came in by tracing or logs, which do not carry the per-category results. See [Which way in carries what](#azure-ai-content-safety). |
| `413`, `429` | As for any gateway: [Limits](gateways.md#limits). |
