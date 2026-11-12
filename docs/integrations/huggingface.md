---
description: Monitor Hugging Face Transformers pipelines with WhiteBoxXAI — register the model, wrap a pipeline so predictions are logged, log batches and generation metrics.
---

# Hugging Face Transformers

`TransformersMonitor` logs predictions from Hugging Face Transformers pipelines (text classification,
sentiment analysis, token classification, question answering, text generation, summarization and
translation) so WhiteBoxXAI can track drift and behaviour over time.

For generative models used as LLMs (chat, RAG, agents), log calls with
[LLM monitoring](../sdk/llm-monitoring.md) instead, which records prompts, completions, tokens and
safety scans.

## Install

```bash
pip install "whitebox-xai-sdk[huggingface]"
```

## Register, then wrap a pipeline

```python
import os

from transformers import pipeline
from whiteboxxai import WhiteBoxXAI
from whiteboxxai.integrations.transformers import TransformersMonitor, wrap_transformers_pipeline

client = WhiteBoxXAI(api_key=os.environ["WHITEBOX_API_KEY"])

classifier = pipeline("sentiment-analysis", model="distilbert-base-uncased-finetuned-sst-2-english")

registered = client.models.register(
    name="support-ticket-sentiment",
    model_type="neural_network",
    framework="transformers",
    version="1.0.0",
    features=["text"],
    baseline_metrics={},          # add metrics you measured on your own test set
)

monitor = TransformersMonitor(client, pipeline=classifier, model_id=registered["id"])
classifier = wrap_transformers_pipeline(classifier, monitor)

classifier("The update fixed my issue, thanks!")    # logged
```

!!! tip "Or let the monitor register it"
    `monitor.register_from_model(name="support-ticket-sentiment", features=["text"])` registers the
    model and returns its id. `model_type` defaults to `llm` for generation tasks (text generation,
    translation, summarization) and `neural_network` otherwise, and `version` defaults to `1.0.0`. Pass
    `baseline_metrics={...}` once you have measured some on your own test set.

## Log without wrapping

```python
result = monitor.predict("Where is my refund?")      # runs the pipeline and logs
monitor.log_prediction_transformers(input_text=text, prediction=result)
monitor.log_batch_transformers(inputs=texts, predictions=results)
```

## Text generation

For generation pipelines, record what was generated and how long it took:

```python
monitor.log_generation_metrics(
    prompt=prompt,
    generated_text=output,
    num_tokens=num_new_tokens,
    generation_time=elapsed_seconds,
)
```

## Tips

- Log the input text the model actually received, and no more. Mask personal data first; see
  [Privacy helpers](../sdk/index.md#privacy-helpers).
- Pin the model revision you load, and register a new WhiteBoxXAI version when you change it, so drift
  and audits refer to the model that produced them.

## Next steps

- [LLM monitoring](../sdk/llm-monitoring.md) for chat and RAG applications
- [Python SDK](../sdk/index.md)
- Example: [`transformers_example.py`](https://github.com/AgentaFlow/whitebox-python-sdk/blob/main/examples/transformers_example.py) in the public SDK repository
