# Hugging Face Transformers - Quick Reference

Quick reference for monitoring Hugging Face Transformers models with WhiteBoxXAI.

## Installation

```bash
pip install whiteboxxai[transformers]
```

## Basic Setup

```python
from transformers import pipeline
from whiteboxxai import WhiteBoxXAI
from whiteboxxai.integrations.transformers import TransformersMonitor

# Initialize
client = WhiteBoxXAI(api_key="your-api-key")
classifier = pipeline("sentiment-analysis")

# Create monitor
monitor = TransformersMonitor(client, pipeline=classifier)
monitor.register_from_model(name="Sentiment Classifier", version="1.0.0")
```

## Common Tasks

### Text Classification

```python
# Single prediction
result = monitor.predict("I love this!", log=True)

# Batch predictions
results = monitor.predict(["Good!", "Bad!", "Okay."], log=True)
```

### Named Entity Recognition

```python
ner = pipeline("ner", aggregation_strategy="simple")
monitor = TransformersMonitor(client, pipeline=ner)
monitor.register_from_model(name="NER Model", task="ner")

entities = monitor.predict("Apple Inc. CEO Tim Cook", log=True)
```

### Question Answering

```python
qa = pipeline("question-answering")
monitor = TransformersMonitor(client, pipeline=qa)
monitor.register_from_model(name="QA Model", task="qa")

result = qa(
    question="What is AI?",
    context="AI stands for Artificial Intelligence."
)
monitor.log_prediction_transformers(
    input_text=f"Q: What is AI?",
    prediction=result
)
```

### Text Generation

```python
generator = pipeline("text-generation", model="gpt2")
monitor = TransformersMonitor(client, pipeline=generator)
monitor.register_from_model(name="GPT-2", task="text-generation")

result = generator("AI is", max_length=50)

monitor.log_generation_metrics(
    prompt="AI is",
    generated_text=result[0]['generated_text'],
    num_tokens=len(result[0]['generated_text'].split())
)
```

## Integration Patterns

### Method 1: Direct Monitoring

```python
monitor = TransformersMonitor(client, pipeline=classifier)
monitor.register_from_model(name="My Model")

result = monitor.predict("Input", log=True)
```

### Method 2: Wrap Pipeline

```python
from whiteboxxai.integrations.transformers import wrap_transformers_pipeline

wrapped = wrap_transformers_pipeline(classifier, monitor)
result = wrapped("Input")  # Auto-logged
```

### Method 3: Pipeline Wrapper Class

```python
from whiteboxxai.integrations.transformers import TransformersPipelineWrapper

wrapper = TransformersPipelineWrapper(
    pipeline=classifier,
    client=client,
    auto_register=True
)
result = wrapper("Input")  # Auto-logged
```

## Advanced Features

### Baseline for Drift Detection

```python
baseline_texts = [
    "Positive example",
    "Negative example",
    "Neutral example"
]
monitor.set_baseline(baseline_texts)
```

### Custom Metadata

```python
result = monitor.predict(
    "Input",
    log=True,
    user_id="user_123",
    source="api"
)
```

### Sampling Rate

```python
monitor = TransformersMonitor(
    client=client,
    pipeline=classifier,
    sampling_rate=0.1  # Log 10%
)
```

### Generation Metrics

```python
import time

start = time.time()
result = generator("Prompt", max_length=100)
duration = time.time() - start

monitor.log_generation_metrics(
    prompt="Prompt",
    generated_text=result[0]['generated_text'],
    num_tokens=50,
    generation_time=duration,
    temperature=0.8
)
```

## Supported Tasks

| Task | Pipeline | Monitor Setup |
|------|----------|---------------|
| Sentiment Analysis | `pipeline("sentiment-analysis")` | `TransformersMonitor(client, pipeline=pipe)` |
| Text Classification | `pipeline("text-classification")` | `TransformersMonitor(client, pipeline=pipe)` |
| NER | `pipeline("ner")` | `TransformersMonitor(client, pipeline=pipe, task="ner")` |
| Question Answering | `pipeline("question-answering")` | `TransformersMonitor(client, pipeline=pipe, task="qa")` |
| Text Generation | `pipeline("text-generation")` | `TransformersMonitor(client, pipeline=pipe, task="text-generation")` |
| Translation | `pipeline("translation_xx_to_yy")` | `TransformersMonitor(client, pipeline=pipe, task="translation")` |
| Summarization | `pipeline("summarization")` | `TransformersMonitor(client, pipeline=pipe, task="summarization")` |

## Model Types

```python
# Classification models → "classification"
monitor.register_from_model(name="Classifier")  # Auto-detects

# Generation models → "generation"
monitor.register_from_model(name="Generator", task="text-generation")

# QA models → "qa"
monitor.register_from_model(name="QA Model", task="question-answering")
```

## Common Pipelines

```python
# Sentiment
pipeline("sentiment-analysis")

# Emotion
pipeline("text-classification", model="bhadresh-savani/distilbert-base-uncased-emotion")

# Zero-shot classification
pipeline("zero-shot-classification")

# NER
pipeline("ner", aggregation_strategy="simple")

# Summarization
pipeline("summarization", model="facebook/bart-large-cnn")

# Translation
pipeline("translation_en_to_fr")

# Text generation
pipeline("text-generation", model="gpt2")
```

## GPU Usage

```python
# Single GPU
classifier = pipeline("sentiment-analysis", device=0)

# Multi-GPU
generator = pipeline(
    "text-generation",
    model="facebook/opt-1.3b",
    device_map="auto"
)
```

## Error Handling

```python
try:
    result = monitor.predict(text, log=True)
except Exception as e:
    print(f"Error: {e}")
    result = None
```

## Performance Tips

1. **Use Batch Processing**
   ```python
   results = monitor.predict(texts, log=True)  # Batch
   ```

2. **Use GPU**
   ```python
   classifier = pipeline("sentiment-analysis", device=0)
   ```

3. **Use Distilled Models**
   ```python
   pipeline("sentiment-analysis", model="distilbert-base-uncased")
   ```

4. **Set Sampling Rate**
   ```python
   TransformersMonitor(client, pipeline=pipe, sampling_rate=0.1)
   ```

5. **Cache Models**
   ```python
   # Models cached in ~/.cache/huggingface by default
   ```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Import error | `pip install transformers torch` |
| GPU not detected | Check CUDA: `torch.cuda.is_available()` |
| Slow predictions | Use batch processing, GPU, or smaller models |
| Memory issues | Use distilled models or quantization |
| Rate limiting | Reduce `sampling_rate` |

## Additional Resources

- Full guide: `sdk/guides/HUGGINGFACE_INTEGRATION.md`
- Examples: `sdk/examples/transformers_example.py`
- API docs: https://docs.whiteboxxai.com
- Transformers docs: https://huggingface.co/docs/transformers
