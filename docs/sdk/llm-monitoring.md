---
description: Log LLM calls to WhiteBoxXAI from any provider — OpenAI, Anthropic Claude, Google Gemini, xAI Grok, Mistral, AWS Bedrock, Azure OpenAI, Hugging Face or local models — with a reusable decorator, safety scans, RAG retrievals, prompt versions and grouped runs.
---

# LLM monitoring (any provider)

WhiteBoxXAI doesn't wrap or proxy your LLM provider. Your application calls the model as it does today,
then logs what happened with `client.llm.log_call`. Because logging is a plain function call, it works
with **any model you can call from Python**:

- OpenAI and Azure OpenAI
- Anthropic Claude, directly or through AWS Bedrock or Google Vertex AI
- Google Gemini
- xAI Grok
- Mistral, Cohere and other hosted APIs
- Meta Llama, Qwen, DeepSeek and other open models on any host
- Hugging Face, Ollama, vLLM or your own fine-tuned model

The `provider` field is free text, so use whatever names your team will recognise in the
[LLM & RAG audit trail](../user-guide/observability.md#llm--rag-audit-trails).

## What to log

```python
client.llm.log_call(
    provider="openai",            # required: any string
    model_name="gpt-4o",          # required
    prompt=prompt,                # required
    latency_ms=640.0,             # required
    completion=answer,
    prompt_tokens=412,
    completion_tokens=96,
    status="success",             # or "failure", "timeout", "rate_limited"
    session_id="conv-1842",       # group a conversation
    user_id="agent-7",            # who or what made the call
    environment="production",
    prompt_version_id=version_id, # link to a governed prompt version
    model_id=model_id,            # link to a registered AI system
)
```

Log what the provider actually returned. Don't estimate token counts; leave fields out when the
provider didn't report them.

**Cost** is calculated from the token counts for model names WhiteBoxXAI has prices for: current and
recent OpenAI, Anthropic Claude, Google Gemini and Azure OpenAI models. For other models the call is
logged in full and cost is left blank rather than guessed.

Each logged call is one [Governance Event](../get-started/governance-events.md). Wrap a multi-step run
in `governance_run()` to share events across up to 100 records.

## A decorator for any LLM call

Write one small wrapper and reuse it for every provider:

```python
import functools
import os
import time

from whiteboxxai import WhiteBoxXAI

wbx = WhiteBoxXAI(api_key=os.environ["WHITEBOX_API_KEY"])


def governed_llm(provider, model_name, extract):
    """Log every call to the wrapped function. `extract(result)` returns
    (completion_text, prompt_tokens, completion_tokens)."""
    def decorator(fn):
        @functools.wraps(fn)
        def wrapper(prompt, *args, **kwargs):
            start = time.perf_counter()
            try:
                result = fn(prompt, *args, **kwargs)
            except Exception as err:
                wbx.llm.log_call(provider=provider, model_name=model_name, prompt=prompt,
                                 latency_ms=(time.perf_counter() - start) * 1000,
                                 status="failure", error_message=str(err))
                raise
            completion, p_tok, c_tok = extract(result)
            wbx.llm.log_call(provider=provider, model_name=model_name, prompt=prompt,
                             completion=completion, prompt_tokens=p_tok, completion_tokens=c_tok,
                             latency_ms=(time.perf_counter() - start) * 1000)
            return result
        return wrapper
    return decorator
```

In production, log from a background thread or queue so logging never slows a user-facing response,
and consider [offline mode](offline-mode.md) so records survive a network blip.

## Provider recipes

Each recipe shows the provider call and how to pull out the completion and token counts.

=== "OpenAI"

    ```python
    from openai import OpenAI

    openai_client = OpenAI()

    @governed_llm("openai", "gpt-4o", lambda r: (
        r.choices[0].message.content, r.usage.prompt_tokens, r.usage.completion_tokens))
    def ask(prompt):
        return openai_client.chat.completions.create(
            model="gpt-4o", messages=[{"role": "user", "content": prompt}])
    ```

=== "Anthropic Claude"

    ```python
    import anthropic

    claude = anthropic.Anthropic()

    @governed_llm("anthropic", "claude-sonnet-4-6", lambda r: (
        r.content[0].text, r.usage.input_tokens, r.usage.output_tokens))
    def ask(prompt):
        return claude.messages.create(
            model="claude-sonnet-4-6", max_tokens=1024,
            messages=[{"role": "user", "content": prompt}])
    ```

=== "Google Gemini"

    ```python
    from google import genai

    gemini = genai.Client()

    @governed_llm("google", "gemini-2.5-pro", lambda r: (
        r.text, r.usage_metadata.prompt_token_count, r.usage_metadata.candidates_token_count))
    def ask(prompt):
        return gemini.models.generate_content(model="gemini-2.5-pro", contents=prompt)
    ```

=== "xAI Grok"

    Grok's API is OpenAI-compatible, so the OpenAI SDK works with xAI's base URL:

    ```python
    import os

    from openai import OpenAI

    grok = OpenAI(base_url="https://api.x.ai/v1", api_key=os.environ["XAI_API_KEY"])

    @governed_llm("xai", "grok-4", lambda r: (
        r.choices[0].message.content, r.usage.prompt_tokens, r.usage.completion_tokens))
    def ask(prompt):
        return grok.chat.completions.create(
            model="grok-4", messages=[{"role": "user", "content": prompt}])
    ```

=== "Mistral"

    ```python
    import os

    from mistralai import Mistral

    mistral = Mistral(api_key=os.environ["MISTRAL_API_KEY"])

    @governed_llm("mistral", "mistral-large-latest", lambda r: (
        r.choices[0].message.content, r.usage.prompt_tokens, r.usage.completion_tokens))
    def ask(prompt):
        return mistral.chat.complete(
            model="mistral-large-latest", messages=[{"role": "user", "content": prompt}])
    ```

=== "AWS Bedrock"

    The Bedrock Converse API works the same way for every model Bedrock hosts:

    ```python
    import boto3

    bedrock = boto3.client("bedrock-runtime")
    MODEL = "anthropic.claude-sonnet-4-6"   # or a Llama, Mistral, Nova or other Bedrock model id

    @governed_llm("bedrock", MODEL, lambda r: (
        r["output"]["message"]["content"][0]["text"],
        r["usage"]["inputTokens"], r["usage"]["outputTokens"]))
    def ask(prompt):
        return bedrock.converse(
            modelId=MODEL, messages=[{"role": "user", "content": [{"text": prompt}]}])
    ```

=== "Azure OpenAI"

    ```python
    import os

    from openai import AzureOpenAI

    azure = AzureOpenAI(azure_endpoint=os.environ["AZURE_OPENAI_ENDPOINT"],
                        api_key=os.environ["AZURE_OPENAI_API_KEY"], api_version="2024-10-21")

    @governed_llm("azure", "azure-gpt-4", lambda r: (
        r.choices[0].message.content, r.usage.prompt_tokens, r.usage.completion_tokens))
    def ask(prompt):
        return azure.chat.completions.create(
            model="your-deployment-name", messages=[{"role": "user", "content": prompt}])
    ```

=== "Local (Ollama)"

    ```python
    import ollama

    @governed_llm("ollama", "llama3.1", lambda r: (
        r["message"]["content"], r.get("prompt_eval_count"), r.get("eval_count")))
    def ask(prompt):
        return ollama.chat(model="llama3.1", messages=[{"role": "user", "content": prompt}])
    ```

Model ids change often; use the ones your provider currently offers. Provider SDK details are in each
provider's own documentation.

### Microsoft Foundry

For `azure-ai-inference`, `AzureOpenAI` and `AIProjectClient`, wrap the client instead of logging each call:

```python
from whiteboxxai.integrations.foundry import FoundryMonitor, wrap_foundry_client

monitor = FoundryMonitor(client, model_id="<AI system id>", project_name="claims-bot")
chat = wrap_foundry_client(ChatCompletionsClient(endpoint, credential), monitor)
```

It records latency, tokens, the prompt and response (personal data masked by default) and Azure AI Content Safety's
result, and never changes what your call returns or raises. See [Microsoft Foundry](../integrations/microsoft-foundry.md#wrap-a-client-in-python)
for what it wraps and what it does not.

## LangChain and LangGraph

If your application uses LangChain, the callback handler logs every LLM call for you, whichever chat
model class you use, and groups each root chain run into shared Governance Events:

```python
from whiteboxxai.integrations.langchain import LangChainMonitor, WhiteBoxXAICallbackHandler

monitor = LangChainMonitor(client, application_name="support-assistant", model_id=app_id)
handler = WhiteBoxXAICallbackHandler(monitor)
chain.invoke({"question": question}, config={"callbacks": [handler]})
```

See [LangChain](../integrations/langchain.md) and [LangChain Multi-Agent](../integrations/langchain-multi-agent.md).

## Safety scans

Check prompts or completions for toxicity, personal data and harmful content:

```python
result = client.safety.analyze(content=answer, content_type="completion")
print(result["safety_status"])
```

Toxicity categories, 12 PII types and harmful content are scored, and flagged categories are recorded
as found. `analyze_batch(contents=[...])` takes up to 100 items. Each scan is a Governance Event.

## RAG retrievals

```python
client.rag.log_retrieval(
    query=question,
    results=[{"document_id": "kb-17", "rank": 1, "score": 0.83}],
    top_k=5,
    retrieval_method="vector",
    ground_truth_ids=["kb-17"],     # optional: enables precision, recall, MRR and NDCG
    answer=answer,
)
```

Retrieval quality metrics are only computed when you supply ground-truth document ids. Without them,
they're reported as not computed.

## Group a run

```python
from whiteboxxai import governance_run

with governance_run():
    docs = retrieve(question)
    client.rag.log_retrieval(query=question, results=docs, top_k=5, retrieval_method="vector")
    answer = ask(build_prompt(question, docs))     # the decorated call logs itself
    client.safety.analyze(content=answer.text if hasattr(answer, "text") else str(answer),
                          content_type="completion")
```

All the records in the block share Governance Events: one per 100 records.

## LLM explainability

`client.llm_xai` analyses a prompt and completion pair: `token_importance`, `attention`,
`prompt_sensitivity`, `counterfactuals` and `debug_prompt`, or several at once on a logged call with
`batch_analyze(llm_log_id, ...)`. Each response states the method it used; keep that with any number you
pass on.

### RAG attribution and ungrounded claims

To check whether an answer is supported by the chunks that were retrieved for it, send the query, the
answer and the chunks:

```python
result = client.llm_xai.rag_attribution(
    query="Can a customer ask us to delete their data?",
    answer=answer_text,
    contexts=[
        {"document_id": "policy-7", "title": "Data retention policy", "text": chunk_1, "chunk_index": 0},
        {"document_id": "faq-12", "title": "Privacy FAQ", "text": chunk_2, "chunk_index": 1},
    ],
    model_id=model_id,          # optional: files the analysis under an AI system
)

for claim in result["ungrounded_claims"]:
    print("No supporting chunk:", claim)
print(result["plain_language_verdict"], result["analysis_method"])
```

Each chunk needs `text`; `document_id`, `title`, `chunk_index` and `retrieval_score` are optional. You
can pass `llm_log_id` or `retrieval_log_id` instead of the text, to analyse a call you already logged.
The result has `chunk_attributions` (how much each chunk contributed), `claim_groundings` (each claim
and its best supporting chunk), `ungrounded_claims`, and `faithfulness`, `context_relevance` and
`answer_relevance` scores. An ungrounded claim is one no retrieved chunk supports; it isn't proof the
claim is false. `arag_attribution` is the async version.

Every explanation is stored. `client.llm_xai.get_unified_explanation(explanation_id)` returns it in one
shape whatever method made it: `model_type` (`generative_llm` or `rag_pipeline`), `explanation_method`,
`plain_language_verdict`, `measured` and `attribution_data`.

!!! note "Next SDK release"
    `rag_attribution` and `get_unified_explanation` are in the current development version of the
    SDK and aren't in the published 1.3.0 yet. Until they ship, call
    `POST /api/v1/llm-xai/rag-attribution` and
    `GET /api/v1/llm-xai/explanations/{explanation_id}/unified` directly; see the
    [REST API Reference](api-reference.md#llm-explainability).

The dashboard shows the same analyses in
[Observability → LLM & RAG](../user-guide/observability.md#attribution--explainability).

## Next steps

- [Evaluations & Prompt Versions](../user-guide/evaluations.md): govern prompts and check changes for regressions
- [Multi-Agent Monitoring](multi-agent-monitoring.md): trace agent workflows
- [Observability](../user-guide/observability.md): see it all in the dashboard
