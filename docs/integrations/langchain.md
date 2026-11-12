---
description: Monitor LangChain and LangGraph applications with WhiteBoxXAI — a callback handler that logs every LLM call, chain run, tool call and retrieval, and groups each root run into shared Governance Events.
---

# LangChain

The WhiteBoxXAI callback handler records what a LangChain or LangGraph application does: each LLM
call (model, prompt, completion, tokens, latency), chain runs, tool calls and errors. It works with any
LangChain chat model, so OpenAI, Anthropic, Gemini, Mistral, Bedrock, Ollama and the rest are all
covered.

Each root run's records are sent as one
[governance run](../get-started/governance-events.md#grouped-runs-one-event-per-100-records), so a run
with a dozen LLM calls costs one Governance Event, not twelve.

## Install

```bash
pip install "whitebox-xai-sdk[langchain]" langchain-openai     # or langchain-anthropic, langchain-google-genai, ...
```

## Set up the handler

Register the application as an LLM system once, then create a monitor for it and a callback handler:

```python
import os

from whiteboxxai import WhiteBoxXAI
from whiteboxxai.integrations.langchain import LangChainMonitor, WhiteBoxXAICallbackHandler

client = WhiteBoxXAI(api_key=os.environ["WHITEBOX_API_KEY"])

app = client.models.register(
    name="support-assistant",
    model_type="llm",
    framework="langchain",
    version="1.4.0",
    features=["question"],
    baseline_metrics={},
)

monitor = LangChainMonitor(client, application_name="support-assistant", model_id=app["id"])
handler = WhiteBoxXAICallbackHandler(monitor)
```

!!! tip "Or let the monitor register it"
    `monitor.register_application(name="support-assistant", version="1.4.0", features=["question"])`
    registers the application as an `llm` system and returns its id, and
    `monitor.create_callback_handler()` then returns the same handler as
    `WhiteBoxXAICallbackHandler(monitor)`. If the application isn't registered yet,
    `create_callback_handler()` registers it with defaults (version `1.0.0`, one input named `input`),
    so register it yourself when you want real versions and input names. The name may contain only
    letters, digits, spaces, `-` and `_`; the API rejects other characters.

## Use it

Pass the handler in the run config. It works with chains, chat models, agents and LangGraph graphs:

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI

prompt = ChatPromptTemplate.from_messages([
    ("system", "You answer customer questions using the policy excerpts provided."),
    ("human", "{question}"),
])
chain = prompt | ChatOpenAI(model="gpt-4o")

answer = chain.invoke(
    {"question": "Can I get a refund after 30 days?"},
    config={"callbacks": [handler]},
)
```

Swap `ChatOpenAI` for `ChatAnthropic`, `ChatGoogleGenerativeAI`, `ChatMistralAI`, `ChatBedrock`,
`ChatOllama` or any other LangChain chat model; logging is the same.

For a LangGraph graph or agent:

```python
result = graph.invoke({"messages": [("user", question)]}, config={"callbacks": [handler]})
```

For multi-agent systems where you want agents, tasks and agent-to-agent interactions recorded as a
workflow, see [LangChain Multi-Agent](langchain-multi-agent.md).

## What's recorded

| Event | Recorded as |
| --- | --- |
| LLM call | An LLM log: provider, model, prompt, completion, tokens, latency, status |
| Chain or graph run | A chain execution on the registered application, with inputs, outputs and duration |
| Tool call | The tool name, input, output and duration |
| Errors | The failure, on the call or run it belongs to |

LLM calls appear in the [LLM & RAG audit trail](../user-guide/observability.md#llm--rag-audit-trails).

## Log retrievals

If your RAG chain retrieves documents, log the retrieval so retrieval quality is tracked:

```python
monitor.log_rag_retrieval(
    query=question,
    documents=[{"document_id": d.metadata["id"], "content": d.page_content} for d in docs],
    num_retrieved=len(docs),
    relevance_scores=scores,
    ground_truth_ids=["kb-17"],    # optional: enables precision, recall, MRR and NDCG
    answer=answer.content,
)
```

## Add safety scans and prompt versions

```python
client.safety.analyze(content=answer.content, content_type="completion")
```

Governed prompt versions are managed with `client.prompts`; link the calls they produce by logging
with `prompt_version_id`. See [Evaluations & Prompt Versions](../user-guide/evaluations.md).

## Troubleshooting

**Nothing appears.** Check the handler is in `config={"callbacks": [...]}` on the `invoke` call (or
attached to the model), and that `WHITEBOX_API_KEY` is set in the process running the chain.

**Costs are blank.** Cost is calculated only for model names WhiteBoxXAI has prices for. See
[LLM monitoring](../sdk/llm-monitoring.md#what-to-log).

## See also

- [LLM monitoring (any provider)](../sdk/llm-monitoring.md)
- [LangChain Multi-Agent](langchain-multi-agent.md)
- Example: [`langchain_example.py`](https://github.com/AgentaFlow/whitebox-python-sdk/blob/main/examples/langchain_example.py) in the public SDK repository
