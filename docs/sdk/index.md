# WhiteBoxXAI SDK Developer Documentation

Complete developer guide for the WhiteBoxXAI Python SDK.

---

## Table of Contents

1. [SDK Overview](#sdk-overview)
2. [Architecture](#architecture)
3. [Core Components](#core-components)
4. [API Reference](#api-reference)
5. [Framework Integrations](#framework-integrations)
6. [Advanced Usage](#advanced-usage)
7. [Error Handling](#error-handling)
8. [Performance Optimization](#performance-optimization)
9. [Security & Privacy](#security--privacy)
10. [Extending the SDK](#extending-the-sdk)

---

## SDK Overview

### Purpose

The WhiteBoxXAI SDK provides a lightweight Python interface for integrating AI observability into your machine learning applications. It handles:

- Model registration and metadata management
- Prediction logging and monitoring
- Data drift detection
- Privacy-preserving data handling
- Automatic retries and error recovery
- Local caching for performance

### Design Principles

1. **Minimal Overhead** - < 5% performance impact on model inference
2. **Framework Agnostic** - Works with any ML framework via base API
3. **Privacy First** - Built-in PII detection and masking
4. **Developer Friendly** - Intuitive API with sensible defaults
5. **Production Ready** - Robust error handling, retries, caching

### Installation

```bash
# Base SDK
pip install whitebox-xai-sdk

# With framework support
pip install whitebox-xai-sdk[sklearn]     # Scikit-learn
pip install whitebox-xai-sdk[pytorch]     # PyTorch
pip install whitebox-xai-sdk[tensorflow]  # TensorFlow
pip install whitebox-xai-sdk[xgboost]     # XGBoost
pip install whitebox-xai-sdk[lightgbm]    # LightGBM
pip install whitebox-xai-sdk[huggingface] # Hugging Face Transformers
pip install whitebox-xai-sdk[langchain]   # LangChain
pip install whitebox-xai-sdk[all]         # All integrations
```

---

## Architecture

### Component Diagram

```
┌─────────────────────────────────────────┐
│         User Application                │
│  (Scikit-learn, PyTorch, etc.)         │
└──────────────┬──────────────────────────┘
               │
               ↓
┌─────────────────────────────────────────┐
│      Framework Integrations             │
│  (SklearnMonitor, TorchMonitor)        │
└──────────────┬──────────────────────────┘
               │
               ↓
┌─────────────────────────────────────────┐
│         ModelMonitor                    │
│  (Core monitoring logic)                │
└──────────────┬──────────────────────────┘
               │
               ↓
┌─────────────────────────────────────────┐
│         WhiteBoxXAI Client                │
│  (API communication)                    │
└──────────────┬──────────────────────────┘
               │
               ↓
┌─────────────────────────────────────────┐
│      Middleware Layer                   │
│  (Caching, Retries, Privacy)           │
└──────────────┬──────────────────────────┘
               │
               ↓
┌─────────────────────────────────────────┐
│      WhiteBoxXAI REST API                 │
│  (Backend service)                      │
└─────────────────────────────────────────┘
```

### Class Hierarchy

```python
WhiteBoxXAI               # Main client
├── Config              # Configuration management
├── APIClient           # HTTP client wrapper
└── ModelMonitor        # Core monitoring
    ├── SklearnMonitor  # Scikit-learn integration
    ├── TorchMonitor    # PyTorch integration
    └── (Future integrations)

Utils
├── PIIDetector         # Privacy protection
├── DataMasker          # Data masking
├── TTLCache            # Local caching
└── RetryHandler        # Retry logic
```

---

## Core Components

### 1. WhiteBoxXAI Client

Main entry point for SDK functionality.

**Initialization:**

```python
from whiteboxxai import WhiteBoxXAI

# From API key
client = WhiteBoxXAI(api_key="your-api-key")

# From environment variable
import os
os.environ["WHITEBOXXAI_API_KEY"] = "your-api-key"
client = WhiteBoxXAI()

# With custom config
from whiteboxxai import Config

config = Config(
    api_key="your-api-key",
    api_url="https://api.whiteboxxai.com",
    timeout=30,
    max_retries=3
)
client = WhiteBoxXAI(config=config)
```

**Authentication:**

`api_key` (or the `WHITEBOXXAI_API_KEY` environment variable) is sent as
`Authorization: Bearer <api_key>`. Two credential types work:

- **A dedicated API key** (`wbx_live_...`) — scoped, revocable, and doesn't expire on a fixed
  schedule. This is what you want for the SDK.
- **A login JWT** from `POST /api/v1/auth/login` — convenient for a quick interactive script,
  but it expires in about 30 minutes and carries a human account's full privileges. Don't put
  one in a pipeline.

To obtain a key:

1. Log in to the WhiteBoxXAI web application
2. Navigate to **Profile → API Keys**
3. Generate a new key with the scopes you need and an optional expiry
4. Copy it immediately — it's shown only once — and store it in your secrets manager

Key creation is admin-only, since a key is an organization-wide credential. Full reference:
[API Keys](/account/api-keys/).

**Note:** API keys authenticate on their own. Enabling 2FA on your account protects
interactive logins and does not affect existing SDK integrations.

**Methods:**

```python
# Models
client.models.register(name, version, model_type, **kwargs)
client.models.get(model_id)
client.models.list(**filters)
client.models.update(model_id, **kwargs)
client.models.update_status(model_id, new_status)
client.models.update_baseline(model_id, ...)
client.models.get_versions(model_name)
client.models.get_latest(model_name)
client.models.archive(model_id)
client.models.restore(model_id)
client.models.delete(model_id)

# Predictions
client.predictions.log(model_id, input_data, output_data, **kwargs)
client.predictions.log_batch(model_id, predictions)
client.predictions.get(prediction_id)
client.predictions.query(model_id, start_time, end_time, limit, offset)
client.predictions.get_stats(model_id, ...)
client.predictions.get_recent(model_id, limit=10)

# Drift
client.drift.detect(model_id, ...)
client.drift.create_report(model_id, window_size=1000)
client.drift.get_reports(model_id, limit=10, skip=0)
client.drift.get_report(model_id, report_id)
client.drift.get_trend(model_id, days=7)

# Explanations
client.explanations.generate(model_id, instance, method=None, ...)
client.explanations.get(explanation_id)

# Fairness
client.fairness.audit(model_id, ...)
client.fairness.get_audit(audit_id)
client.fairness.list_audits(...)
client.fairness.get_bias_history(model_id, days=30)
client.fairness.get_metric_history(model_id, metric_type, days=30)
client.fairness.get_latest_audit(model_id)

# Alerts
client.alerts.create(name, alert_type, severity, conditions, model_id=None, ...)
client.alerts.list(model_id=None, alert_type=None, is_active=None, ...)
client.alerts.get_rule(rule_id)
client.alerts.update_rule(rule_id, **kwargs)
client.alerts.delete_rule(rule_id)
client.alerts.evaluate_rule(rule_id, metric_values=None, ...)
client.alerts.list_instances(rule_id=None, model_id=None, status=None, ...)
client.alerts.get_instance(alert_id)
client.alerts.acknowledge(alert_id, user_id, notes=None)
client.alerts.resolve(alert_id, user_id, resolution_notes=None)
```

Every method has an `a`-prefixed async twin — `client.models.aregister()`,
`client.predictions.alog()`, `client.drift.aget_reports()`, and so on.

!!! note "There is no `client.metrics`"
    Metrics are read through the [REST API](/sdk/api-reference/#metrics) directly. The SDK
    client exposes `models`, `predictions`, `explanations`, `drift`, `fairness`, and `alerts`.

### 2. ModelMonitor

Core monitoring logic for model predictions.

**Initialization:**

```python
from whiteboxxai import WhiteBoxXAI, ModelMonitor

client = WhiteBoxXAI(api_key="your-api-key")

monitor = ModelMonitor(
    client,
    model_id=123,              # Existing model
    model_name=None,           # Or look up / register by name
    auto_explain=False,        # Generate an explanation for every prediction
    sampling_rate=1.0,         # Log this fraction of predictions
    buffer_size=None,          # Buffer locally and send as batches
)

# Or register a new model
monitor = ModelMonitor(client)
model_id = monitor.register_model(
    name="my_model",
    version="1.0.0",
    model_type="classification",
    framework="sklearn"
)
```

**Core Methods:**

```python
# Register model
model_id = monitor.register_model(
    name: str,
    model_type: str,  # "classification", "regression", etc.
    framework: str = None,
    version: str = "1.0.0",
    features: List[str] = None,
    target: str = None,
    baseline_metrics: Dict = None,
    tags: List[str] = None,
    metadata: Dict = None
) -> str

# Log single prediction. Returns the prediction data, or None if it was
# buffered or skipped by sampling.
monitor.log_prediction(
    inputs: Any,
    output: Any,
    explain: bool = None,          # Defaults to the monitor's auto_explain
    metadata: Dict[str, Any] = None
) -> Optional[Dict[str, Any]]

# Log batch predictions. Each item takes inputs/output or
# input_data/output_data keys.
monitor.log_batch(predictions: List[Dict]) -> Dict[str, Any]

# Send buffered predictions now
monitor.flush() -> Optional[Dict[str, Any]]
await monitor.aflush()

# Convenience reads
monitor.get_prediction_count() -> int
monitor.get_drift_reports(limit: int = 10, skip: int = 0) -> List[Dict]

# Set baseline data
monitor.set_baseline(data: np.ndarray)

# Drift detection
monitor.detect_drift(...)
```

`log_prediction(explain=True)` really does generate an explanation — in versions before
1.0.0 the flag was accepted and silently dropped.

`monitor.create_alert_rule()` and `monitor.get_active_alerts()` call the [Alerts
API](/sdk/api-reference/#alerts) — see [Managing Alerts](/user-guide/#managing-alerts) for the
dashboard equivalent.

```python
monitor.create_alert_rule(
    metric="accuracy",
    threshold=0.85,
    condition="below",
    severity="high",   # required — there's no default, pass it via kwargs
)

active = monitor.get_active_alerts()
```

This is a convenience wrapper over `client.alerts.create()` with a simpler, single-condition
signature — for multi-condition rules, or full control over notification channels and
throttling, call `client.alerts.create()` directly instead.

**Local buffering:**

Set `buffer_size` to accumulate predictions locally and send them as batches, which costs far
fewer API calls than one request per prediction:

```python
monitor = ModelMonitor(client, model_id=123, buffer_size=100)

for row in stream:
    monitor.log_prediction(inputs=row.features, output=row.prediction)
    # Flushes automatically every 100 predictions

monitor.flush()  # Send whatever's left
```

Prefer the context manager, which flushes on exit even if the block raises:

```python
with ModelMonitor(client, model_id=123, buffer_size=100) as monitor:
    for row in stream:
        monitor.log_prediction(inputs=row.features, output=row.prediction)
# Buffer flushed here
```

As a backstop, the monitor also registers a weakly-referenced `atexit` hook that makes a
best-effort flush when the interpreter shuts down, so a process that exits without an explicit
`flush()` doesn't silently lose its buffer. Treat it as a safety net, not a strategy — an
explicit `flush()` or `with` block is what you should write.

**Configuration:**

```python
# Sampling rate (log N% of predictions)
monitor.sampling_rate = 0.1  # 10%

# Explanation generation for every logged prediction
monitor.auto_explain = True
```

### 3. Framework Integrations

#### Scikit-learn

```python
from whiteboxxai.integrations.sklearn import SklearnMonitor

# Initialize
sklearn_monitor = SklearnMonitor(
    client=client,
    model=trained_model,
    model_id="existing-id"  # Optional
)

# Auto-register from trained model
model_id = sklearn_monitor.register_from_model(
    name="sklearn_model",
    model_type="classification",
    X_train=X_train,  # For baseline
    y_train=y_train
)

# Wrap model for automatic monitoring
monitored_model = sklearn_monitor.wrap_model(trained_model)

# Use normally - predictions auto-logged
predictions = monitored_model.predict(X_test)
probabilities = monitored_model.predict_proba(X_test)

# Manual logging
sklearn_monitor.log_prediction_from_model(
    X=input_features,
    y_pred=prediction,
    y_prob=probabilities  # Optional
)
```

**Features:**
- Automatic feature extraction from model
- Support for pipelines and feature transformers
- Baseline calculation from training data
- Model metadata extraction (n_features, classes, etc.)

#### PyTorch

```python
from whiteboxxai.integrations.pytorch import TorchMonitor

# Initialize
torch_monitor = TorchMonitor(
    client=client,
    model=pytorch_model,
    device="cuda"  # or "cpu"
)

# Register
model_id = torch_monitor.register_from_model(
    name="torch_model",
    model_type="classification",
    input_shape=(1, 28, 28),
    output_classes=10
)

# Wrap model
monitored_model = torch_monitor.wrap_model(pytorch_model)

# Forward pass with automatic logging
outputs = monitored_model(inputs)

# Or manual logging
with torch.no_grad():
    outputs = model(inputs)
    torch_monitor.log_prediction_from_tensor(
        inputs=inputs,
        outputs=outputs
    )
```

**Features:**
- GPU/CPU compatibility
- Batch prediction logging
- Automatic shape inference
- Hook-based monitoring (optional)

### 4. Configuration Management

```python
from whiteboxxai import Config

config = Config(
    # API settings
    api_key="your-api-key",
    api_url="https://api.whiteboxxai.com",
    api_version="v1",

    # Network settings
    timeout=30,
    max_retries=3,
    retry_backoff=2.0,

    # Caching
    enable_cache=True,
    cache_ttl=300,
    cache_max_size=1000,

    # Privacy
    mask_pii=True,
    allowed_fields=["feature1", "feature2"],

    # Logging
    log_level="INFO",
    log_to_file=False,
    log_file_path="/var/log/whiteboxxai.log",

    # Performance
    batch_size=100,
    sampling_rate=1.0,
    async_logging=True
)

client = WhiteBoxXAI(config=config)
```

**Environment Variables:**

```bash
# Required
WHITEBOXXAI_API_KEY=your-api-key

# Optional
WHITEBOXXAI_API_URL=https://api.whiteboxxai.com
WHITEBOXXAI_TIMEOUT=30
WHITEBOXXAI_MAX_RETRIES=3
WHITEBOXXAI_CACHE_TTL=300
WHITEBOXXAI_SAMPLING_RATE=0.1
WHITEBOXXAI_LOG_LEVEL=INFO
```

---

## API Reference

### Models API

#### register_model()

Register a new model for monitoring.

```python
model_id = client.models.register(
    name: str,                      # Required
    version: str,                   # Required
    model_type: str,                # Required: "classification", "regression", etc.
    framework: str = None,          # Optional: "sklearn", "pytorch", etc.
    description: str = None,
    features: List[str] = None,
    target: str = None,
    baseline_metrics: Dict = None,
    tags: List[str] = None,
    metadata: Dict = None
) -> str
```

**Returns:** Model ID (UUID string)

**Example:**

```python
model_id = client.models.register(
    name="fraud_detector",
    version="2.1.0",
    model_type="classification",
    framework="xgboost",
    description="Detects fraudulent transactions",
    features=[
        "transaction_amount",
        "merchant_category",
        "time_of_day",
        "customer_age"
    ],
    target="is_fraud",
    baseline_metrics={
        "accuracy": 0.94,
        "precision": 0.91,
        "recall": 0.89,
        "f1_score": 0.90,
        "auc_roc": 0.96
    },
    tags=["production", "fraud", "high-priority"],
    metadata={
        "owner": "data-science-team",
        "training_date": "2025-11-15",
        "model_path": "s3://models/fraud-v2.1.0"
    }
)
```

#### get_model()

Retrieve model information.

```python
model = client.models.get(model_id: str) -> Dict
```

**Returns:**
```python
{
    "id": "model-uuid",
    "name": "fraud_detector",
    "version": "2.1.0",
    "model_type": "classification",
    "framework": "xgboost",
    "status": "active",
    "created_at": "2025-12-01T10:00:00Z",
    "features": [...],
    "baseline_metrics": {...},
    "tags": [...],
    "metadata": {...}
}
```

#### update_model()

Update model metadata.

```python
client.models.update(
    model_id: str,
    name: str = None,
    description: str = None,
    tags: List[str] = None,
    metadata: Dict = None,
    status: str = None  # "active", "inactive", "deprecated"
)
```

#### list_models()

List all models with optional filters.

```python
models = client.models.list(
    tags: List[str] = None,
    model_type: str = None,
    status: str = None,
    limit: int = 100,
    offset: int = 0
) -> List[Dict]
```

### Predictions API

#### log_prediction()

Log a single prediction.

```python
prediction_id = client.predictions.log(
    model_id: str,
    inputs: Dict[str, Any],
    output: Dict[str, Any],
    metadata: Dict[str, Any] = None,
    timestamp: datetime = None,
    prediction_id: str = None  # Optional custom ID
) -> str
```

**Example:**

```python
prediction_id = client.predictions.log(
    model_id="model-uuid",
    input_data={
        "transaction_amount": 250.00,
        "merchant_category": "electronics",
        "time_of_day": 14,
        "customer_age": 35
    },
    output_data={
        "prediction": "legitimate",
        "fraud_probability": 0.08,
        "confidence": 0.92
    },
    metadata={
        "transaction_id": "txn_123456",
        "customer_id": "cust_789012",
        "ip_address": "masked",
        "device": "mobile"
    }
)
```

#### log_batch()

Log multiple predictions efficiently.

```python
prediction_ids = client.predictions.log_batch(
    model_id: str,
    predictions: List[Dict]
) -> List[str]
```

**Prediction format:**
```python
predictions = [
    {
        "inputs": {...},
        "output": {...},
        "metadata": {...},
        "timestamp": "2025-12-05T10:30:00Z"  # Optional
    },
    # ... up to 1000 predictions
]
```

**Example:**

```python
predictions = []
for transaction in transactions:
    pred = model.predict(transaction.features)
    predictions.append({
        "inputs": transaction.to_dict(),
        "output": {"prediction": pred, "probability": model.predict_proba(transaction.features)[0][1]},
        "metadata": {"transaction_id": transaction.id}
    })

prediction_ids = client.predictions.log_batch(
    model_id="model-uuid",
    predictions=predictions
)
```

### Metrics API

There is no `client.metrics` resource. Read metrics through the REST API — see
[Metrics](/sdk/api-reference/#metrics) in the API reference — or use the monitoring dashboards.

For drift and fairness history, which is what most metric queries are actually after, the SDK
does have first-class support:

```python
# Drift over time
client.drift.get_trend(model_id="model-uuid", days=7)
client.drift.get_reports(model_id="model-uuid", limit=10)

# Fairness over time
client.fairness.get_bias_history(model_id="model-uuid", days=30)
client.fairness.get_metric_history(
    model_id="model-uuid",
    metric_type="demographic_parity",
    days=30,
)
```

To call the metrics endpoints directly with the client's authenticated session:

```python
metrics = client.request("GET", "/api/v1/metrics", params={"model_id": "model-uuid"})
```

### Drift Detection API

#### detect_drift()

Run drift detection on new data.

```python
drift_result = client.drift.detect(
    model_id: str,
    data: Union[pd.DataFrame, np.ndarray],
    feature_names: List[str] = None,
    reference: str = "baseline"  # "baseline" or "recent"
) -> Dict
```

**Returns:**

```python
{
    "overall_score": 0.15,
    "severity": "medium",
    "drifted_features": [
        {
            "feature": "transaction_amount",
            "drift_score": 0.22,
            "severity": "high",
            "test": "ks_test",
            "p_value": 0.001
        }
    ],
    "timestamp": "2025-12-05T14:30:00Z"
}
```

### Explanations API

#### generate()

Generate a SHAP or LIME explanation for a specific input instance.

```python
explanation = client.explanations.generate(
    model_id: str,
    instance: Dict[str, Any],       # the feature values to explain
    method: str = None,             # "shap" or "lime" — server default if omitted
    prediction_id: str = None,      # optional association with a logged prediction
    num_features: int = 10,         # 1-100
    num_samples: int = 5000,        # 100-50000
    use_cache: bool = True,         # reuse a cached result for identical (model_id, instance, method)
) -> Dict
```

Blocks until the explanation is ready. An `a`-prefixed async twin, `agenerate()`, exists like
every other SDK method.

**Example:**

```python
explanation = client.explanations.generate(
    model_id="model-uuid",
    instance={
        "transaction_amount": 250.00,
        "merchant_category": "electronics",
        "time_of_day": 14,
        "customer_age": 34
    },
    method="shap"
)

# Returns a plain dict matching the ExplanationResponse schema:
{
    "id": "exp-uuid",
    "model_id": "model-uuid",
    "prediction_id": None,
    "method": "shap",
    "status": "completed",
    "base_value": 0.20,
    "score": 0.65,
    "feature_weights": {
        "transaction_amount": 0.15,
        "merchant_category": 0.10,
        "time_of_day": -0.05,
        "customer_age": 0.25
    },
    "cached": False,
    "computation_time_ms": 842.1
}
```

!!! warning "Field names are `feature_weights` and `score`"
    Not `.feature_contributions` or `.prediction` — and the result is a `dict`, not an object
    with attributes. `explanation["feature_weights"]`, not
    `explanation.feature_contributions`.

#### Other explanation methods

Every method below has an `a`-prefixed async twin.

```python
# Non-blocking: up to 100 instances in one call
client.explanations.generate_bulk(model_id, instances, method=None, parallel=True)
# -> {"explanation_ids": [...], "total": N, "message": "..."}

# Non-blocking: a single instance, poll get() until status != "pending"
client.explanations.generate_async(model_id, instance, method=None, prediction_id=None)
# -> {"explanation_id": "...", "status": "pending", "message": "..."}

client.explanations.get(explanation_id)
client.explanations.list_by_model(model_id, method=None, status=None, limit=50, offset=0)
client.explanations.get_by_prediction(prediction_id)
client.explanations.get_stats(model_id)

# Per-model explanation defaults (replaces any `explanation_config=` kwarg on models.update —
# that kwarg doesn't exist)
client.explanations.set_config(
    model_id, enabled=True, default_method="shap_kernel", auto_explain=False,
    cache_enabled=True, cache_ttl_hours=24,
)
client.explanations.get_config(model_id)
client.explanations.delete_config(model_id)

# Compare 2-20 existing explanations
client.explanations.compare(explanation_ids, comparison_type="feature_importance")
# comparison_type: "feature_importance" | "instance_similarity" | "method_agreement"

# Visualization data (not a rendered plot — chart-ready data for your own UI)
client.explanations.visualize(explanation_id, plot_type, max_features=10)
# plot_type: "waterfall" | "force" | "feature_importance"
client.explanations.visualize_multi(explanation_ids, plot_type, max_features=20)
# plot_type: "summary" | "decision" | "dependence", 1-100 ids
```

!!! note "No global feature-importance endpoint"
    There's no server-side "global importance across all predictions" call. Aggregate it
    yourself from `list_by_model()`. There's also no custom-explainer registration mechanism
    (`BaseExplainer`/`register_explainer`) and no `export_report()` — build a PDF/HTML export
    through [Audit & Explanation Reports](/user-guide/reports/) instead, which packages
    explanations alongside everything else in a report.

---

## Framework Integrations

### Adding New Framework Integration

Create a new integration by extending `ModelMonitor`:

```python
from whiteboxxai import ModelMonitor
from typing import Any, Dict, List

class CustomFrameworkMonitor(ModelMonitor):
    """Monitor for CustomFramework models."""

    def __init__(self, client, model, **kwargs):
        super().__init__(client, **kwargs)
        self.model = model

    def register_from_model(
        self,
        name: str,
        model_type: str,
        **kwargs
    ) -> str:
        """Register model by extracting metadata."""
        # Extract features from model
        features = self._extract_features()

        # Extract model metadata
        metadata = self._extract_metadata()

        return self.register_model(
            name=name,
            model_type=model_type,
            framework="customframework",
            features=features,
            metadata=metadata,
            **kwargs
        )

    def _extract_features(self) -> List[str]:
        """Extract feature names from model."""
        # Implementation specific to framework
        return self.model.feature_names_

    def _extract_metadata(self) -> Dict:
        """Extract model metadata."""
        return {
            "n_features": self.model.n_features_,
            "model_params": self.model.get_params()
        }

    def wrap_model(self, model):
        """Wrap model for automatic monitoring."""
        original_predict = model.predict

        def monitored_predict(X):
            result = original_predict(X)
            # Log prediction
            self.log_prediction_from_model(X, result)
            return result

        model.predict = monitored_predict
        return model

    def log_prediction_from_model(
        self,
        X: Any,
        y_pred: Any,
        **kwargs
    ):
        """Log prediction in framework-specific format."""
        # Convert to SDK format
        inputs = self._convert_inputs(X)
        output = self._convert_output(y_pred)

        return self.log_prediction(
            inputs=inputs,
            output=output,
            **kwargs
        )
```

---

## Advanced Usage

### Async Operations

Use async methods for non-blocking operations:

```python
import asyncio
from whiteboxxai import WhiteBoxXAI

client = WhiteBoxXAI(api_key="your-api-key")

async def log_predictions_async():
    tasks = []
    for prediction in predictions:
        task = client.predictions.alog(
            model_id="model-uuid",
            input_data=prediction["input_data"],
            output_data=prediction["output_data"]
        )
        tasks.append(task)

    results = await asyncio.gather(*tasks)
    return results

# Run
prediction_ids = asyncio.run(log_predictions_async())
```

### Custom Sampling Strategy

Implement custom sampling logic:

```python
from whiteboxxai import ModelMonitor
import random

class SmartSamplingMonitor(ModelMonitor):
    def log_prediction(self, inputs, output, **kwargs):
        # Sample based on confidence
        confidence = output.get("confidence", 1.0)

        if confidence < 0.6:
            sample_rate = 1.0  # Log all low-confidence
        elif confidence < 0.8:
            sample_rate = 0.5  # Log 50% medium-confidence
        else:
            sample_rate = 0.1  # Log 10% high-confidence

        if random.random() < sample_rate:
            return super().log_prediction(inputs, output, **kwargs)
```

### Streaming Predictions

For high-throughput scenarios:

```python
from whiteboxxai import WhiteBoxXAI, ModelMonitor
from queue import Queue
import threading

class StreamingMonitor:
    def __init__(self, client, model_id, buffer_size=100, flush_interval=10):
        self.monitor = ModelMonitor(client, model_id=model_id)
        self.buffer = Queue(maxsize=buffer_size)
        self.flush_interval = flush_interval
        self._start_flush_thread()

    def log(self, inputs, output, metadata=None):
        """Add prediction to buffer."""
        self.buffer.put({
            "inputs": inputs,
            "output": output,
            "metadata": metadata
        })

    def _start_flush_thread(self):
        """Background thread to flush buffer."""
        def flush():
            while True:
                time.sleep(self.flush_interval)
                self._flush_buffer()

        thread = threading.Thread(target=flush, daemon=True)
        thread.start()

    def _flush_buffer(self):
        """Flush buffer to API."""
        predictions = []
        while not self.buffer.empty() and len(predictions) < 100:
            predictions.append(self.buffer.get())

        if predictions:
            self.monitor.log_batch(predictions)

# Usage
monitor = StreamingMonitor(client, model_id="model-uuid")

for prediction in high_volume_predictions:
    monitor.log(
        inputs=prediction["inputs"],
        output=prediction["output"]
    )
```

---

## Error Handling

### Exception Hierarchy

```python
from whiteboxxai.exceptions import (
    WhiteBoxXAIError,      # Base exception — catch this to catch everything
    APIError,              # API request failed
    AuthenticationError,   # Invalid or expired credential
    ValidationError,       # Invalid input data
    RateLimitError,        # Rate limit exceeded
    NotFoundError,         # Resource does not exist
    ConfigurationError,    # Bad SDK configuration
    IntegrationError,      # Framework integration failure
    CacheError,            # Cache layer failure
)
```

### Exception Metadata

Exceptions carry structured attributes rather than just a message, so you can branch on them
instead of parsing strings:

| Exception | Attributes |
| --- | --- |
| `APIError` | `status_code`, `response`, `request_id` |
| `AuthenticationError` | `status_code` |
| `RateLimitError` | `retry_after` (seconds) |
| `ValidationError` | `fields` (per-field error detail) |

```python
from whiteboxxai.exceptions import RateLimitError, ValidationError, APIError
import time

try:
    client.predictions.log(model_id=model_id, input_data=x, output_data=y)
except RateLimitError as e:
    time.sleep(e.retry_after or 60)
except ValidationError as e:
    for field, problem in e.fields.items():
        print(f"{field}: {problem}")
except APIError as e:
    print(f"{e.status_code} (request {e.request_id})")
```

`request_id` is worth logging — it's what support will ask for.

### Error Handling Patterns

```python
import time

import httpx

from whiteboxxai import WhiteBoxXAI
from whiteboxxai.exceptions import AuthenticationError, RateLimitError

client = WhiteBoxXAI(api_key="your-api-key")

def log_with_retry(input_data, output_data, max_retries=3):
    """Log prediction with custom retry logic."""
    for attempt in range(max_retries):
        try:
            return client.predictions.log(
                model_id="model-uuid",
                input_data=input_data,
                output_data=output_data,
            )
        except AuthenticationError:
            # Don't retry auth errors — the credential won't fix itself
            raise
        except RateLimitError as e:
            time.sleep(e.retry_after or 60)
        except httpx.TransportError:
            # Connection-level failure: exponential backoff
            if attempt < max_retries - 1:
                time.sleep(2 ** attempt)
            else:
                raise
```

!!! tip "Or just let offline mode handle it"
    Connection failures on `predictions.log()`, `predictions.log_batch()`,
    `models.register()`, and `models.update_baseline()` are exactly what [offline
    mode](/sdk/offline-mode/) is for — enable it and the operation is queued and retried for you,
    no retry loop required.

### Graceful Degradation

```python
def monitored_predict(X):
    """Prediction with fallback if monitoring fails."""
    # Always make prediction
    prediction = model.predict(X)

    # Try to log, but don't fail if it doesn't work
    try:
        client.predictions.log(
            model_id="model-uuid",
            input_data=X.to_dict(),
            output_data={"prediction": prediction}
        )
    except Exception as e:
        # Log error but continue
        logger.warning(f"Failed to log prediction: {e}")

    return prediction
```

---

## Performance Optimization

### Caching

Use local caching to reduce API calls:

```python
from whiteboxxai import WhiteBoxXAI, Config

config = Config(
    api_key="your-api-key",
    enable_cache=True,
    cache_ttl=300,  # 5 minutes
    cache_max_size=1000
)

client = WhiteBoxXAI(config=config)

# First call hits API
model = client.models.get("model-uuid")

# Second call within TTL uses cache
model = client.models.get("model-uuid")  # Instant
```

### Batch Operations

Always prefer batch operations:

```python
# ❌ Slow: 100 API calls
for prediction in predictions:
    client.predictions.log(model_id, prediction["inputs"], prediction["output"])

# ✅ Fast: 1 API call
client.predictions.log_batch(model_id, predictions)
```

### Async for I/O-bound Operations

```python
import asyncio

async def process_predictions():
    tasks = [
        client.predictions.alog(...)
        for _ in range(1000)
    ]
    await asyncio.gather(*tasks)

# Much faster than sequential
asyncio.run(process_predictions())
```

---

## Security & Privacy

### PII Detection

Built-in PII detection:

```python
from whiteboxxai.utils import PIIDetector

detector = PIIDetector()

data = {
    "name": "John Doe",
    "email": "john@example.com",
    "ssn": "123-45-6789",
    "amount": 100.0
}

# Detect PII
pii_fields = detector.detect(data)
# Returns: ["name", "email", "ssn"]

# Check if PII present
has_pii = detector.has_pii(data)
# Returns: True
```

### Data Masking

Automatically mask sensitive data:

```python
from whiteboxxai.utils import DataMasker

masker = DataMasker(
    mask_pii=True,
    allowed_fields=["amount", "category"]
)

data = {
    "name": "John Doe",
    "email": "john@example.com",
    "amount": 100.0,
    "category": "electronics"
}

masked = masker.mask(data)
# Returns:
# {
#     "amount": 100.0,
#     "category": "electronics"
# }
```

### Secure Configuration

Store API keys securely:

```python
# ✅ Use environment variables
import os
os.environ["WHITEBOXXAI_API_KEY"] = "your-api-key"
client = WhiteBoxXAI()

# ✅ Use secrets management
from your_secrets_manager import get_secret
api_key = get_secret("whiteboxxai/api-key")
client = WhiteBoxXAI(api_key=api_key)

# ❌ Don't hardcode
client = WhiteBoxXAI(api_key="sk-1234567890")  # Bad!
```

---

## Extending the SDK

### Custom Middleware

Add custom processing logic:

```python
from whiteboxxai import WhiteBoxXAI

class CustomMiddleware:
    def __init__(self, client):
        self.client = client
        self._wrap_methods()

    def _wrap_methods(self):
        """Wrap client methods with custom logic."""
        original_log = self.client.predictions.log

        def wrapped_log(*args, **kwargs):
            # Pre-processing
            print("Logging prediction...")

            # Call original
            result = original_log(*args, **kwargs)

            # Post-processing
            print(f"Logged: {result}")

            return result

        self.client.predictions.log = wrapped_log

# Usage
client = WhiteBoxXAI(api_key="your-api-key")
middleware = CustomMiddleware(client)

# Now all log calls go through middleware
client.predictions.log(...)
```

### Custom Explainer

Add custom explanation methods:

```python
from whiteboxxai.xai import BaseExplainer

class CustomExplainer(BaseExplainer):
    """Custom explanation method."""

    def explain(self, model, X, **kwargs):
        """Generate custom explanation."""
        # Your explanation logic
        feature_importance = self._calculate_importance(model, X)

        return {
            "method": "custom",
            "feature_contributions": feature_importance,
            "metadata": {...}
        }

    def _calculate_importance(self, model, X):
        """Calculate feature importance."""
        # Implementation
        pass

# Register custom explainer
from whiteboxxai.xai import register_explainer

register_explainer("custom", CustomExplainer)

# Use
explanation = client.explanations.generate(
    prediction_id="pred-uuid",
    method="custom"
)
```

---

## Troubleshooting

### Common Issues

**Issue: "Authentication failed"**
```python
# Check API key
print(os.environ.get("WHITEBOXXAI_API_KEY"))

# Verify key is valid
from whiteboxxai import WhiteBoxXAI
client = WhiteBoxXAI(api_key="your-key")
try:
    client.models.list()
    print("API key valid")
except AuthenticationError:
    print("API key invalid")
```

**Issue: "Model not found"**
```python
# List all models
models = client.models.list()
for model in models:
    print(f"{model['id']}: {model['name']}")

# Check model ID
model_id = "your-model-id"
try:
    model = client.models.get(model_id)
    print(f"Model found: {model['name']}")
except APIError:
    print(f"Model {model_id} not found")
```

**Issue: "Rate limit exceeded"**
```python
from whiteboxxai.exceptions import RateLimitError
import time

try:
    client.predictions.log(...)
except RateLimitError as e:
    print(f"Rate limited. Retry after {e.retry_after}s")
    time.sleep(e.retry_after)
    client.predictions.log(...)  # Retry
```

### Debug Mode

Enable debug logging:

```python
import logging

logging.basicConfig(level=logging.DEBUG)

# Or SDK-specific
from whiteboxxai import Config

config = Config(
    api_key="your-api-key",
    log_level="DEBUG"
)
client = WhiteBoxXAI(config=config)
```

---

## Examples

See `/examples` directory for complete examples:
- `basic_usage.py` - Simple prediction logging
- `sklearn_integration.py` - Scikit-learn integration
- `pytorch_integration.py` - PyTorch integration
- `batch_logging.py` - High-volume batch logging
- `custom_sampling.py` - Custom sampling strategies
- `async_operations.py` - Async prediction logging

---

*Last Updated: December 30, 2025*
*SDK Version: 1.0.0*
*Recent Updates: Added API key generation instructions with 2FA note*
