---
description: Hands-on tutorial — govern an LLM application with WhiteBoxXAI: log calls from any provider, scan answers for safety, version the prompt, score it against a golden dataset, and check a prompt change for regressions.
---

# Tutorial: govern an LLM application

You'll take a small customer-support assistant and give it the governance a reviewer expects: every
call logged, answers scanned for toxicity and personal data, the prompt under version control, and a
regression check before a prompt change ships. It takes about 40 minutes.

**You need:** Python 3.10 or later, a WhiteBoxXAI [API key](../account/api-keys.md) in
`WHITEBOX_API_KEY`, and an API key for any LLM provider. The examples use OpenAI; swap in any provider
from [LLM monitoring](../sdk/llm-monitoring.md#provider-recipes).

```bash
pip install whitebox-xai-sdk openai
```

## 1. Register the application

```python
import os

from whiteboxxai import WhiteBoxXAI

client = WhiteBoxXAI(api_key=os.environ["WHITEBOX_API_KEY"])

app = client.models.register(
    name="support-assistant-tutorial",
    model_type="llm",
    framework="openai",
    version="1.0.0",
    features=["question"],
    baseline_metrics={},
    tags=["tutorial"],
)
```

## 2. Put the prompt under version control

```python
template = client.prompts.create_template(name="support-answer", description="Tutorial prompt")
v1 = client.prompts.create_version(
    template["id"],
    system_message="You are a support assistant. Answer only from the policy text given.",
    user_message_template="Policy:\n{policy}\n\nQuestion: {question}",
    model_name="gpt-4o-mini",
    temperature=0.2,
    version_tag="v1",
)
```

Open **Observability → Evaluations → Prompts**: the prompt and its first version are there. Versions
can't be edited later; a change is always a new version.

## 3. Log every call, and scan the answers

```python
import time

from openai import OpenAI
from whiteboxxai import governance_run

llm = OpenAI()
POLICY = "Refunds are available within 30 days of purchase with a receipt."


def answer(question: str) -> str:
    prompt = f"Policy:\n{POLICY}\n\nQuestion: {question}"
    with governance_run():                       # the call and its scan share one event
        start = time.perf_counter()
        r = llm.chat.completions.create(
            model="gpt-4o-mini",
            temperature=0.2,
            messages=[{"role": "system", "content": "You are a support assistant. Answer only from the policy text given."},
                      {"role": "user", "content": prompt}],
        )
        text = r.choices[0].message.content
        client.llm.log_call(
            provider="openai", model_name="gpt-4o-mini", prompt=prompt, completion=text,
            latency_ms=(time.perf_counter() - start) * 1000,
            prompt_tokens=r.usage.prompt_tokens, completion_tokens=r.usage.completion_tokens,
            prompt_version_id=v1["id"], model_id=app["id"], session_id="tutorial",
        )
        client.safety.analyze(content=text, content_type="completion", model_id=app["id"])
    return text


for q in ["Can I get a refund after two weeks?", "Can I get a refund without a receipt?"]:
    print(answer(q))
```

Open **Observability → LLM & RAG** for the support assistant: the conversation log, token usage, spend
and safety results. Each call is linked to prompt version `v1`.

## 4. Build a golden dataset

A golden dataset records inputs, the answer you expect, and the answer the model actually gave.
WhiteBoxXAI scores what's stored; it never calls your model itself.

In **Observability → Evaluations → Golden datasets**:

1. **New dataset**: name it `refund-questions`.
2. **Add a row** for each case, with the **Input**, an **Expected output** and the **Stored model
   output** from step 3. For example:

    | Input | Expected output | Stored model output |
    | --- | --- | --- |
    | `{"question": "Can I get a refund after two weeks?"}` | `{"must_contain": ["30 days"]}` | `{"answer": "<the answer you got>"}` |
    | `{"question": "Can I get a refund without a receipt?"}` | `{"must_contain": ["receipt"]}` | `{"answer": "<the answer you got>"}` |

## 5. Score the prompt version

**Runs → Score a prompt version**: choose `support-answer · v1`, the `refund-questions` dataset, and
**Required phrases** as how to score. The run reports **N of M evaluated rows passed**, lists any rows
**Not evaluated**, and records how it was scored.

## 6. Change the prompt and check for regressions

1. Save a new version, `v2`, with a different system message, for example a friendlier tone.
2. Run the questions again with `v2` (pass `prompt_version_id=v2["id"]`) and add the new answers to a
   copy of the dataset rows.
3. **Score a prompt version** for `v2` on the same dataset.
4. **Check a prompt change for regressions**: baseline run `v1`, candidate run `v2`.

The verdict is **No regression found**, **Regression found** (with the cases that got worse), or
**Cannot tell** if too few rows were judged in both runs. Only deploy `v2` when you're satisfied:

```python
client.prompts.advance_stage(template["id"], v2["id"], target_stage="verify")
client.prompts.deploy(template["id"], v2["id"])     # refused without a passed verification
```

## 7. Clean up

```python
client.models.archive(app["id"])
```

## What you've built

| Evidence | Where |
| --- | --- |
| Every LLM call, with tokens, latency and the prompt version that produced it | Observability → LLM & RAG |
| Safety scan results for every answer | Observability → LLM & RAG → Safety |
| A versioned, immutable prompt history | Observability → Evaluations → Prompts |
| Scored evaluation runs and a regression verdict | Observability → Evaluations → Runs |

Next: [Connect an AI assistant over MCP](tutorial-agent-mcp.md).
