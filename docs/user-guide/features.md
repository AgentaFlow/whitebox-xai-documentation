# WhiteBoxXAI Features

Comprehensive documentation for all WhiteBoxXAI features and capabilities.

---

## Table of Contents

1. [Audit & Explanation Reports](#audit--explanation-reports)
2. [Model Registry](#model-registry)
3. [Prediction Logging](#prediction-logging)
4. [Metrics Dashboard](#metrics-dashboard)
5. [Drift Detection](#drift-detection)
6. [Explainability Engine](#explainability-engine)
7. [Bias Detection](#bias-detection)
8. [LLM Monitoring](#llm-monitoring)
9. [Alert System](#alert-system)
10. [Reporting](#reporting)
11. [Compliance](#compliance)

---

## Audit & Explanation Reports

Need audit evidence or a plain-English explanation of a decision, not the SDK reference?
Start here instead: **[Audit & Explanation Reports](/user-guide/reports/)** covers report
categories, the dashboard walkthrough, and why every number in a report traces back to real
computed data — not a template.

For the explainability computation itself (SHAP, LIME, feature importance) see
[Explainability Engine](#explainability-engine) below; for fairness audits see
[Bias Detection](#bias-detection).

---

## Model Registry

The Model Registry is your central hub for managing all ML models.

### Overview

**Purpose:**
- Centralize model information
- Track model versions
- Monitor model lifecycle
- Document model metadata

**Key Capabilities:**
- Register unlimited models (plan-dependent)
- Version tracking by name
- Tags and metadata
- Model archival

### Registering Models

#### Via Web UI

1. Navigate to **Models** page
2. Click **Register Model** button
3. Fill in model details:

**Required Fields:**
- **Name** - Descriptive model name
- **Version** - Semantic version (e.g., "1.0.0")
- **Type** - Classification, Regression, Clustering, LLM

**Recommended Fields:**
- **Description** - Model purpose and use case
- **Framework** - ML library used (scikit-learn, PyTorch, etc.)
- **Features** - Input feature names
- **Target** - Output variable name
- **Baseline Metrics** - Performance on test set

**Optional Fields:**
- **Tags** - Labels for organization (e.g., "production", "finance")
- **Training Data** - Size, date, source
- **Model File** - Link to model artifact
- **Documentation** - Link to detailed docs

#### Via SDK

```python
from whiteboxxai import WhiteBoxXAI

client = WhiteBoxXAI()

model = client.models.register(
    name="Customer Churn Predictor",
    model_type="classification",
    version="2.1.0",
    framework="xgboost",
    description="Predicts customer churn risk using behavioral data",
    features=[
        "account_age_days",
        "total_purchases",
        "avg_purchase_value",
        "days_since_last_purchase",
        "customer_service_calls",
        "satisfaction_score"
    ],
    target_variable="will_churn",
    baseline_metrics={
        "accuracy": 0.89,
        "precision": 0.85,
        "recall": 0.82,
        "f1_score": 0.835,
        "auc_roc": 0.93
    },
    tags=["production", "customer-retention", "high-priority"],
)

print(f"Model registered with ID: {model['id']}")
```

!!! note "Every SDK call returns a plain `dict`"
    Not an object with attributes. It's `model["id"]`, not `model.id`, throughout this page —
    that applies to every SDK response shown below, not just this one.

There's also `auto_detect_git=True`, which reads the current commit, branch, and working-tree
state from the repository you're registering from and attaches them automatically — the
Git-side half of the traceability [GitHub Integration](/integrations/github/) provides. Add
`require_clean_git=True` to refuse registration with uncommitted changes.

#### Via REST API

```bash
curl -X POST https://api.whiteboxxai.com/api/v1/models \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Customer Churn Predictor",
    "version": "2.1.0",
    "model_type": "classification",
    "features": ["account_age_days", "total_purchases"],
    "baseline_metrics": {"accuracy": 0.89}
  }'
```

### Model Versioning

Version is a free-text string you choose — semantic versioning (`2.1.0`) is a sane convention,
not something the platform enforces. Register a new version under the same `name` to track it
as part of that model's history:

```python
# Register a new version under the same name
client.models.register(
    name="Customer Churn Predictor",
    model_type="classification",
    version="2.2.0",
    features=[...],
    baseline_metrics={...},
)

# List every version registered under that name, newest first
versions = client.models.get_versions(model_name="Customer Churn Predictor")
for v in versions:
    print(f"v{v['version']} - {v['status']} - {v['created_at']}")

# Or just the most recent one
latest = client.models.get_latest(model_name="Customer Churn Predictor")
```

There's no built-in model-to-model comparison endpoint — pull each version's metrics and
compare them yourself, or view them side by side on the **Models** page in the dashboard.

### Model Metadata

**Tracked Automatically:**
- Registration date
- Last updated
- Prediction count
- Last prediction timestamp
- Active alerts
- Drift status

**Update model fields:**

`update()` is a partial update — only the fields you pass change. The full field list is
`description`, `status`, `features`, `baseline_metrics`, `target_variable`,
`model_framework`, `fairness_attributes`, `protected_groups`, and `tags`; there's no free-form
`metadata` dict beyond those.

```python
client.models.update(
    model_id="model-uuid",
    description="Retrained on Q4 data; approved by risk committee 2026-08-20.",
    tags=["production", "compliance-approved"],
)
```

If you need to track something not on that list — an owner, a model card URL, a risk
rating — put it in your own system of record, or as free text in `description`, or as a
[custom metric](#custom-metrics) if it's numeric and you want it on the dashboard.

### Model Status Lifecycle

**Statuses:** `active`, `deprecated`, `archived`. There's no separate "draft" or "staging"
status — a model is either active and monitored, or it isn't.

```python
# Update status
client.models.update_status(model_id="model-uuid", new_status="deprecated")

# Archive a model (soft delete — data is preserved, monitoring stops)
client.models.archive(model_id="old-model-uuid")

# Bring it back
client.models.restore(model_id="old-model-uuid")
```

### Model Organization

**Tags:**
```python
# Set (replaces the existing tag list)
client.models.update(model_id="model-uuid", tags=["production", "high-priority"])

# Search by tags
prod_models = client.models.list(tags=["production"])
```

---

## Prediction Logging

Track all model predictions for monitoring and analysis.

### Overview

**Purpose:**
- Monitor prediction patterns
- Calculate performance metrics
- Detect drift
- Enable explainability
- Audit trail for compliance

**What to Log:**
- Input features (`input_data`)
- Model output — prediction, probability, whatever your model returns (`output_data`)
- Metadata (optional)

!!! note "No ground-truth label update yet"
    There's no endpoint today for attaching an actual outcome to a prediction after the fact
    (a "here's what really happened" update). Accuracy and other metrics that need ground
    truth work from data you log yourself, joined outside the platform, until that ships.

### Logging Methods

#### 1. Manual Logging

Log individual predictions explicitly:

```python
from whiteboxxai import WhiteBoxXAI

client = WhiteBoxXAI()

# After your model predicts
features = {
    "age": 35,
    "income": 75000,
    "credit_score": 720,
    "debt_ratio": 0.35
}

prediction = model.predict([list(features.values())])[0]
probability = model.predict_proba([list(features.values())])[0][1]

# Log to WhiteBoxXAI
response = client.predictions.log(
    model_id="model-uuid",
    input_data=features,
    output_data={
        "prediction": int(prediction),
        "probability": float(probability)
    },
    metadata={
        "customer_id": "CUST-12345",
        "application_id": "APP-67890",
        "channel": "web"
    }
)

print(f"Logged: {response['prediction_id']}")
```

#### 2. `ModelMonitor` — the higher-level path

For most integrations, `ModelMonitor` is less code than calling `client.predictions.log()`
directly — it holds the model ID, and adds optional local buffering and sampling. See [SDK
Documentation](/sdk/#2-modelmonitor) for the full API.

```python
from whiteboxxai import WhiteBoxXAI, ModelMonitor

client = WhiteBoxXAI()
monitor = ModelMonitor(client, model_id="model-uuid")

prediction = model.predict([list(features.values())])[0]
probability = model.predict_proba([list(features.values())])[0][1]

monitor.log_prediction(
    inputs=features,
    output={"prediction": int(prediction), "probability": float(probability)},
)
```

Wrap it in a decorator with `monitor_model()` if you'd rather not touch the call site at all:

```python
from whiteboxxai.decorators import monitor_model

@monitor_model(monitor, input_keys=["features"])
def predict_loan_approval(features):
    prediction = model.predict([features])[0]
    probability = model.predict_proba([features])[0][1]
    return {"prediction": int(prediction), "probability": float(probability)}

# Logging happens automatically on every call
result = predict_loan_approval(features=[35, 75000, 720, 0.35])
```

#### 3. Batch Logging

Log multiple predictions in one call — each item needs `input_data`/`output_data`, matching
the single-prediction shape:

```python
predictions = []

for customer in customers:
    features = extract_features(customer)
    pred = model.predict([features])
    prob = model.predict_proba([features])[0][1]

    predictions.append({
        "input_data": features,
        "output_data": {
            "prediction": int(pred[0]),
            "probability": float(prob)
        },
        "metadata": {"customer_id": customer.id}
    })

# Log batch (up to 1,000 per call)
response = client.predictions.log_batch(
    model_id="model-uuid",
    predictions=predictions
)

print(f"Logged {response['logged']} of {response['total']} ({response['failed']} failed)")
```

#### 4. Framework Integration

**Scikit-learn:**
```python
from whiteboxxai.integrations.sklearn import SklearnMonitor

sklearn_monitor = SklearnMonitor(client=client, model=my_sklearn_model, model_id="model-uuid")
monitored_model = sklearn_monitor.wrap_model(my_sklearn_model)

# Use like normal — logging is automatic
predictions = monitored_model.predict(X_test)
probabilities = monitored_model.predict_proba(X_test)
```

**PyTorch:**
```python
from whiteboxxai.integrations.pytorch import TorchMonitor

torch_monitor = TorchMonitor(client=client, model=pytorch_model)
monitored_model = torch_monitor.wrap_model(pytorch_model)

outputs = monitored_model(inputs)  # Forward pass logs automatically
```

See [SDK Documentation](/sdk/#3-framework-integrations) for `register_from_model()` and the
full framework-integration reference.

### Sampling

Set `sampling_rate` on the monitor rather than on the model — it's a client-side decision
about how much of your own traffic to send, not a server-side model setting:

```python
# Log 10% of predictions, chosen randomly
monitor = ModelMonitor(client, model_id="model-uuid", sampling_rate=0.1)
```

For anything more targeted than uniform random sampling — stratified by confidence, higher
during business hours, always log the ones you'd actually want to review — decide in your own
code whether to call `log_prediction()` for a given prediction; there's no server-side
stratified-sampling config to configure instead.

```python
# Example: always log low-confidence predictions, sample the rest
if probability > 0.9 or probability < 0.1:
    should_log = True
else:
    should_log = random.random() < 0.05

if should_log:
    monitor.log_prediction(inputs=features, output=output)
```

**Recommendations:**
- **< 1k/day:** 100% (log all)
- **1k-10k/day:** 10-50%
- **10k-100k/day:** 1-10%
- **> 100k/day:** 0.1-1%

### Metadata Best Practices

**Do Include:**
- Request IDs (for tracing)
- User IDs (non-PII)
- Environment (production/staging)
- A/B test variant
- Feature flags
- Device/platform info

**Don't Include:**
- PII (names, emails, addresses)
- Sensitive data (SSN, passwords)
- Large objects (>1KB)

**Example:**
```python
metadata={
    "request_id": "req-123456",
    "user_id": "usr-789012",  # Anonymized
    "environment": "production",
    "experiment": "model-v2-test",
    "variant": "treatment",
    "platform": "ios",
    "app_version": "3.2.1",
    "region": "us-west"
}
```

---

## Metrics Dashboard

Real-time visibility into model performance.

### Overview

**Purpose:**
- Monitor model health
- Track performance trends
- Identify issues quickly
- Make data-driven decisions

**Key Metrics:**
- Performance (accuracy, precision, recall)
- Volume (predictions/day)
- Latency (response time)
- Errors (failure rate)

### Dashboard Components

#### 1. Overview Cards

**High-level KPIs:**
- Total Models
- Active Models
- Predictions Today
- Active Alerts
- Average Accuracy
- Drift Detections

**Real-time updates** via WebSocket (every 5 seconds)

#### 2. Time Series Charts

**Prediction Volume:**
- Line chart showing predictions over time
- Granularity: Hourly, Daily, Weekly
- Compare to previous period
- Identify traffic patterns

**Accuracy Trend:**
- Track accuracy changes
- Baseline comparison
- Confidence intervals
- Alert thresholds marked

**Multi-metric View:**
- Overlay multiple metrics
- Compare precision vs recall
- Identify trade-offs

#### 3. Model Performance Table

| Model | Accuracy | Predictions | Drift | Alerts | Last Prediction |
|-------|----------|-------------|-------|--------|-----------------|
| Fraud Detector v2 | 0.89 ↓ | 15.2k | ⚠️ Medium | 2 | 2 min ago |
| Churn Predictor | 0.91 ↑ | 8.5k | ✅ Low | 0 | 5 min ago |

**Actions:**
- Click model to view details
- Sort by any column
- Filter by status/drift/alerts

### Model-Specific Metrics

#### Classification Metrics

**Binary Classification:**
- **Accuracy** - % correct predictions
- **Precision** - True positives / (True positives + False positives)
- **Recall** - True positives / (True positives + False negatives)
- **F1 Score** - Harmonic mean of precision and recall
- **AUC-ROC** - Area under ROC curve
- **AUC-PR** - Area under Precision-Recall curve
- **Log Loss** - Logarithmic loss

**Multi-class Classification:**
- Macro-averaged metrics
- Micro-averaged metrics
- Per-class metrics
- Confusion matrix

**Confusion Matrix:**
```
              Predicted
              No   Yes
Actual  No   TN    FP
        Yes  FN    TP
```

**Thresholds:**
- Adjust decision threshold
- Optimize for precision or recall
- View threshold impact

#### Regression Metrics

- **MAE** - Mean Absolute Error
- **MSE** - Mean Squared Error
- **RMSE** - Root Mean Squared Error
- **R² Score** - Coefficient of determination
- **MAPE** - Mean Absolute Percentage Error
- **Median AE** - Median Absolute Error

**Visualizations:**
- Predicted vs Actual scatter plot
- Residual plot
- Error distribution histogram

#### LLM Metrics

- **Token Usage** - Input/output tokens
- **Cost** - Total spend, cost per request
- **Latency** - Response time (p50, p95, p99)
- **Toxicity** - Safety scores
- **RAG Quality** - Retrieval/answer quality

### Custom Metrics

`metric_type` and `metric_name` are free text — log any business-specific number against a
model, not just the built-in performance metrics:

```python
client.metrics.create(
    model_id="model-uuid",
    metric_type="business",
    metric_name="revenue_impact",
    metric_value=12500.50,
)

# Or several at once (max 100 per call)
client.metrics.create_batch(
    model_id="model-uuid",
    metrics=[
        {"metric_type": "business", "metric_name": "revenue_impact", "metric_value": 12500.50},
        {"metric_type": "business", "metric_name": "cost_savings", "metric_value": 3200.00},
    ],
)

# Read them back
metrics = client.metrics.get_model_metrics(
    model_id="model-uuid",
    metric_type="business",
    start_date="2026-07-01",
    end_date="2026-07-31",
)
```

**Common Custom Metrics:**
- Revenue impact
- Cost savings
- Customer satisfaction
- Processing time
- Business conversion rate

### Metric Aggregations

```python
# Aggregated over a period — "daily", "weekly", etc.
client.metrics.aggregate(model_id="model-uuid", period="daily", metric_type="accuracy")

# Trend and rolling-window statistics
client.metrics.trend(model_id="model-uuid", metric_type="accuracy", lookback_days=30)
client.metrics.rolling(model_id="model-uuid", metric_type="accuracy", window_days=7)

# Raw time series over a required date range
client.metrics.timeseries(
    model_id="model-uuid",
    metric_type="accuracy",
    start_date="2026-07-01",
    end_date="2026-07-31",
)

# Everything for a model in one call
client.metrics.summary(model_id="model-uuid", days=30)
```

### Alerts on Metrics

Set thresholds to get notified:

```python
client.alerts.create(
    name="Low Accuracy Alert",
    alert_type="performance",
    severity="high",
    model_id="model-uuid",
    conditions=[
        {"metric_name": "accuracy", "operator": "lt", "threshold": 0.85, "window_minutes": 60}
    ],
)
```

---

## Drift Detection

Automatically detect when your data or model behavior changes.

### Overview

**Purpose:**
- Identify when model needs retraining
- Catch data quality issues
- Maintain prediction accuracy
- Compliance with model governance

**Types of Drift:**
1. **Data Drift** - Input distribution changes
2. **Concept Drift** - Feature-target relationship changes
3. **Prediction Drift** - Output distribution changes

### Data Drift

**Definition:**
Changes in input feature distributions compared to training data.

**Example:**
Income distribution shifted from mean $50k to $75k

**Detection Methods:**

**Continuous Features:**
- **Kolmogorov-Smirnov (KS) Test** - Compares distributions
- **Population Stability Index (PSI)** - Industry standard

**Categorical Features:**
- **Chi-squared Test** - Compares frequency distributions
- **Jensen-Shannon (JS) Divergence** - Symmetric KL divergence

### Configuring drift detection

Drift configuration is a REST-only surface today — there's no `client.drift_config.*` in the
SDK yet, so this is `curl` rather than Python:

```bash
# Create a default config for a model
curl -X POST https://api.whiteboxxai.com/api/v1/drift-config/{model_id}/default \
  -H "Authorization: Bearer $WHITEBOXXAI_API_KEY"

# Or the fastest path: apply a sensitivity preset directly
curl -X POST https://api.whiteboxxai.com/api/v1/drift-config/{model_id}/sensitivity/medium \
  -H "Authorization: Bearer $WHITEBOXXAI_API_KEY"
```

Three presets ship — `low`, `medium` (recommended default), `high` — each setting the KS,
chi-squared, JS, and PSI thresholds together along with the concept-drift and severity
thresholds, rather than tuning each statistical test individually:

```bash
GET /api/v1/drift-config/presets                # all three, with their actual threshold values
GET /api/v1/drift-config/presets/{preset_name}   # one preset
```

For anything more specific than a preset, `PUT /api/v1/drift-config/{model_id}` takes the full
`feature_configs`, `concept_drift_config`, `alert_config`, `schedule_config`, and
`threshold_config` objects — see the response from `GET /api/v1/drift-config/{model_id}` for
the exact shape a given model currently has.

**Drift Score Interpretation:** the `medium` preset's severity thresholds are a reasonable
default reading — roughly 0.0–0.1 no meaningful drift, 0.1–0.3 worth investigating, 0.3+
action required — but the exact cutoffs are configurable per preset and per model rather than
fixed platform-wide.

### Concept Drift

**Definition:**
Changes in the relationship between features and target — the same input now predicts a
different outcome than it used to.

**Example:**
Credit score of 650 used to have a 20% default rate, now has 10%.

**Detection approach:** tracked via `concept_drift_config` on the same drift configuration
above — performance-degradation-based (comparing recent vs. historical accuracy) and
distribution-based methods, including ADWIN, are available as part of that config rather than
as a separate endpoint.

### Prediction Drift

**Definition:**
Changes in the model's output distribution, independent of whether the inputs changed —
useful for catching a model that's started behaving differently even when the input
population looks the same.

**Example:**
Fraud-rate predictions increased from 5% to 15% of traffic.

### Drift Detection Dashboard

**Drift Overview Page:**
- Models with drift
- Recent drift events
- Drift severity distribution
- Timeline of detections

**Per-Model Drift View:**
- Overall drift score
- Per-feature drift scores
- Distribution comparisons
- Statistical test results
- Recommended actions

### Drift Reports

**Detailed Report Includes:**

**Summary:**
- Overall drift score
- Number of drifted features
- Severity classification
- Detection date and time

**Per-Feature Analysis:**
- Individual drift scores
- Statistical test p-values
- Distribution visualizations
- Before/after statistics

**Visualizations:**
- Distribution overlays (training vs production)
- Time series of drift scores
- Heatmap of feature drift over time

**Recommendations:**
- Which features drifted most
- Potential causes
- Suggested actions

**Example Report:**
```
Drift Detection Report
Model: Fraud Detection v2
Date: 2025-12-05

Overall Drift Score: 0.18 (Medium)

Drifted Features:
1. transaction_amount (High drift: 0.25)
   - Training: mean=$125, std=$50
   - Production: mean=$180, std=$75
   - Interpretation: Customers making larger purchases

2. merchant_category (Medium drift: 0.15)
   - New categories appearing: "crypto", "nft"
   - Training categories decreased: "travel"

3. time_of_day (Low drift: 0.08)
   - Slight shift toward evening transactions

Non-drifted Features:
- customer_age: 0.02
- account_age: 0.03

Recommendations:
✓ Retrain model with recent data (3+ months)
✓ Investigate new merchant categories
✓ Consider adding temporal features
✓ Update baseline after retraining
```

### Responding to Drift

**Workflow:**

**1. Detection → Alert sent**

**2. Investigation:**
```python
# List recent drift reports, most recent first
reports = client.drift.get_reports(model_id="model-uuid", limit=10)
latest = reports[0]

# Per-feature statistics live inside the report
for feature in latest["feature_drifts"]:
    print(f"{feature['feature_name']}: {feature['drift_score']} ({feature['severity']})")

# Trend over time, rather than a single snapshot
trend = client.drift.get_trend(model_id="model-uuid", days=30)

# Or run a fresh, ad-hoc check without waiting for the next scheduled one
result = client.drift.detect(model_id="model-uuid", window_size=1000)
```

Compare distributions visually on the model's **Drift** tab in the dashboard — there's no SDK
method that hands back a chart object; `feature_drifts` above is the raw statistics behind it.

**3. Action:**
- **Low drift:** Monitor, no action yet
- **Medium drift:** Plan retraining within 1-2 weeks
- **High drift:** Immediate retraining or model rollback

**4. Retraining:**
```python
# Register the retrained version under the same name
client.models.register(
    name="Fraud Detection",
    model_type="classification",
    version="2.3.0",
    features=[...],
    baseline_metrics={...},  # New baselines
    tags=["retrained-drift-response"],
)

# Or, refreshing the same model's baseline after retraining in place
client.models.update_baseline(
    model_id="model-uuid",
    baseline_metrics={"accuracy": 0.94, "precision": 0.91},
)
```

### Drift Configuration Best Practices

**Detection Frequency:**
- **High-volume models:** Daily
- **Medium-volume:** Weekly
- **Low-volume:** Monthly

**Thresholds:**
- **Start conservative:** 0.2 (avoid false alarms)
- **Tune down:** 0.1-0.15 (after baseline established)
- **Critical models:** 0.05 (very sensitive)

**Reference Period:**
- **Training data:** For long-term drift
- **Rolling window (30d):** For recent trends
- **Both:** Most comprehensive

---

## Explainability Engine

Understand why your models make specific predictions.

!!! tip "Looking for an explainability report, not the raw computation?"
    See [Audit & Explanation Reports](/user-guide/reports/) for how these explanations get
    packaged into something you can hand to a stakeholder or auditor.

### Overview

**Purpose:**
- Debug model decisions
- Build trust with stakeholders
- Regulatory compliance (right to explanation)
- Identify biases
- Model improvement insights

**Supported Methods:**
- SHAP (SHapley Additive exPlanations)
- LIME (Local Interpretable Model-agnostic Explanations)

!!! note "Full method reference lives with the SDK"
    This section is the concepts and use cases. For the complete, verified method list —
    `generate_bulk()`, `compare()`, `visualize()`/`visualize_multi()`, `set_config()`, and
    every parameter — see [SDK Documentation](/sdk/#explanations-api).

### SHAP Explanations

**What is SHAP?**

Based on game theory (Shapley values), SHAP provides:
- Theoretically sound explanations
- Local and global interpretability
- Consistent feature attributions
- Additive property

**How it Works:**

For each prediction:
1. Calculate the contribution of each feature
2. Base value (average prediction) + feature contributions = final prediction
3. Positive contribution = pushes the prediction higher
4. Negative contribution = pushes it lower

**Generate a SHAP explanation:**

```python
explanation = client.explanations.generate(
    model_id="model-uuid",
    instance={"income": 75000, "credit_score": 780, "debt_ratio": 0.2, "age": 34},
    method="shap",
)

# Access results — this is a dict, not an object with attributes
print(f"Base value: {explanation['base_value']}")
print(f"Score: {explanation['score']}")

for feature, weight in explanation["feature_weights"].items():
    print(f"{feature}: {weight:+.3f}")
```

**Waterfall Chart** — shows cumulative feature contributions:

```
Base Value: 0.20
+ income (high): +0.15
+ credit_score (excellent): +0.25
+ debt_ratio (low): +0.10
- age (young): -0.05
= Final Prediction: 0.65
```

Get the chart-ready data behind it (not a rendered image — data for your own UI) with
`client.explanations.visualize(explanation_id, plot_type="waterfall")`, or `"force"` for a
force-plot layout of the same contributions.

### LIME Explanations

**What is LIME?**

Creates a simple, interpretable model around a specific prediction:
1. Generate perturbations around the instance
2. Get model predictions for perturbations
3. Train a simple linear model on these
4. Use the linear model's coefficients as the explanation

**When to Use LIME:**
- Faster than SHAP for complex models
- Good for high-dimensional data
- Need quick explanations
- Local understanding is sufficient

**Generate a LIME explanation** — the call is identical to SHAP's, just a different `method`:

```python
explanation = client.explanations.generate(
    model_id="model-uuid",
    instance={"credit_score": 720, "income": 75000, "employment_years": 8, "debt_ratio": 0.35},
    method="lime",
    num_features=10,
)

for feature, weight in explanation["feature_weights"].items():
    direction = "increases" if weight > 0 else "decreases"
    print(f"{feature}: {abs(weight):.3f} {direction} the prediction")
```

**Output Example:**
```
Prediction: Approved (probability: 0.85)

Top Contributing Features:
+ credit_score (720): +0.35 (high score increases approval)
+ income ($75k): +0.28 (sufficient income increases approval)
+ employment_years (8): +0.15 (stable employment increases approval)
- debt_ratio (0.35): -0.12 (moderate debt decreases approval)
```

### Global feature importance

There's no single endpoint that returns "global importance across all predictions" —
aggregate it yourself from explanations you've already generated:

```python
explanations = client.explanations.list_by_model(model_id="model-uuid", method="shap", limit=1000)

totals: dict[str, float] = {}
for exp in explanations:
    for feature, weight in exp["feature_weights"].items():
        totals[feature] = totals.get(feature, 0) + abs(weight)

ranked = sorted(totals.items(), key=lambda item: item[1], reverse=True)
```

Or, for a comparison across a specific set of already-generated explanations rather than a
running total, `client.explanations.compare(explanation_ids, comparison_type="feature_importance")`
does the aggregation server-side.

### Explanation Use Cases

**1. Debugging** — pull recent explanations for a model and look for one relying heavily on a
feature that shouldn't matter:

```python
recent = client.explanations.list_by_model(model_id="model-uuid", limit=100)

for exp in recent:
    top_feature = max(exp["feature_weights"], key=lambda f: abs(exp["feature_weights"][f]))
    if top_feature == "zip_code":  # shouldn't be driving decisions
        print(f"Investigate: {exp['id']} (prediction {exp['prediction_id']})")
```

**2. Contestation** — a customer disputes a denial; generate the explanation behind that
specific prediction and hand it to them (or route it into a [report](/user-guide/reports/) for
a formatted document):

```python
explanation = client.explanations.generate(
    model_id="model-uuid",
    instance=denial_input_data,
    prediction_id=denial_prediction_id,
    method="shap",
)
```

**3. Compliance** — the same call, generated on demand for an audit request. It's
automatically retained and searchable via `list_by_model()`/`get_by_prediction()` — there's no
separate "audit metadata" field to set; the explanation itself *is* the record.

**4. Model improvement** — use the ranked list from [Global feature
importance](#global-feature-importance) above to spot low-importance features that might be
candidates for removal.

### Explanation Configuration

Set per-model defaults — method, auto-generation, caching — with `set_config()`:

```python
client.explanations.set_config(
    model_id="model-uuid",
    default_method="shap_kernel",
    auto_explain=False,     # generate on-demand only
    cache_enabled=True,
    cache_ttl_hours=24,
)
```

### Bulk and async generation

For more than a handful of instances, `generate_bulk()` is non-blocking — you poll for results
rather than waiting on the request:

```python
job = client.explanations.generate_bulk(
    model_id="model-uuid",
    instances=[instance_1, instance_2, instance_3],
    method="shap",
)
# {"explanation_ids": [...], "total": 3, "message": "..."}

for explanation_id in job["explanation_ids"]:
    result = client.explanations.get(explanation_id)
    if result["status"] != "pending":
        print(result["feature_weights"])
```

`generate_async()` is the single-instance equivalent — same non-blocking, poll-with-`get()`
pattern, for one instance at a time.

---

## Bias Detection

Audit models for fairness across demographic groups.

!!! tip "Need a bias audit report you can hand to an auditor?"
    See [Audit & Explanation Reports](/user-guide/reports/) — the fairness results below
    feed directly into the **Bias audit** report category.

### Overview

**Purpose:**
- Ensure fair treatment across groups
- Regulatory compliance (ECOA, Fair Housing Act, etc.)
- Ethical AI practices
- Mitigate reputational risk

**Key Capabilities:**
- Multi-metric fairness assessment
- Protected attribute analysis
- Bias severity classification
- Mitigation recommendations

### Fairness Metrics

**1. Demographic Parity (Statistical Parity)**

**Definition:** Positive prediction rates should be equal across groups.

**Formula:**
P(Ŷ=1 | A=a) = P(Ŷ=1 | A=b) for all groups a, b

**Example:**
```
Loan approval rates should be similar:
- Male: 65%
- Female: 63%
- Ratio: 0.97 (PASS - within 0.8-1.2)
```

**When to Use:** When equal outcomes are the goal

**2. Equal Opportunity**

**Definition:** True positive rates should be equal across groups.

**Formula:**
P(Ŷ=1 | Y=1, A=a) = P(Ŷ=1 | Y=1, A=b)

**Example:**
```
Among qualified applicants, approval rates should be equal:
- Male (qualified): 85% approved
- Female (qualified): 87% approved
- Difference: 0.02 (PASS - < 0.1)
```

**When to Use:** When equal opportunity for qualified individuals matters

**3. Equalized Odds**

**Definition:** Both TPR and FPR should be equal across groups.

**Formula:**
P(Ŷ=1 | Y=y, A=a) = P(Ŷ=1 | Y=y, A=b) for y ∈ {0,1}

**Example:**
```
TPR equal + FPR equal:
- TPR: Male 85%, Female 83% (diff: 0.02 ✓)
- FPR: Male 12%, Female 14% (diff: 0.02 ✓)
- PASS
```

**When to Use:** Strictest criterion, both false positives and false negatives matter

**4. Disparate Impact**

**Definition:** Ratio of positive outcomes between groups.

**Formula:**
Disparate Impact = P(Ŷ=1 | A=unprivileged) / P(Ŷ=1 | A=privileged)

**Legal Standard:** Must be ≥ 0.8 (80% rule in US)

**Example:**
```
Female approval: 52%
Male approval: 65%
DI = 52/65 = 0.80 (PASS - exactly at threshold)
```

### Running Fairness Audits

**Via UI:**

1. Navigate to **Bias & Fairness** page
2. Click **Run Audit**
3. Configure:
   - Select model
   - Choose protected attribute (e.g., gender, race, age)
   - Select reference group (e.g., "male" for gender)
   - Choose metrics to calculate
   - Set time period

4. Click **Run Audit**
5. Review results (~1-3 minutes)

**Via SDK:**

`audit()` isn't a data-source query the way drift detection is — it doesn't fetch predictions
for you. You supply the true labels, predicted labels, and each row's group membership; the
platform computes the fairness metrics from that. A common pattern is pulling logged
predictions first, then joining in whatever demographic data you have:

```python
predictions = client.predictions.query(model_id="model-uuid", start_time="2026-07-01T00:00:00Z")

# predictions.query() gives you what the model predicted, not the actual outcome or
# demographic group — join both in from your own records, keyed by prediction_id
y_pred = [p["output_data"]["prediction"] for p in predictions]
y_true = [your_ground_truth[p["prediction_id"]] for p in predictions]
gender = [your_demographics[p["prediction_id"]]["gender"] for p in predictions]

audit = client.fairness.audit(
    model_id="model-uuid",
    sensitive_attributes=["gender"],
    y_true=y_true,
    y_pred=y_pred,
    group_data={"gender": gender},
    metrics_to_compute=["demographic_parity", "equal_opportunity", "equalized_odds", "disparate_impact"],
)

# This is a dict, not an object — same as every other SDK response
print(f"Fairness score: {audit['overall_fairness_score']:.2f}")
print(f"Bias detected: {audit['bias_detected']}")

for metric_name, result in audit["results"].items():
    print(f"{metric_name}: {result['score']:.3f} ({'PASS' if result['passed'] else 'FAIL'})")

for rec in audit["recommendations"]:
    print(f"- {rec}")
```

Pass more than one attribute in `sensitive_attributes` — and a matching key in `group_data` for
each — to audit several protected attributes in one call, rather than needing separate calls
per attribute.

### Understanding Audit Results

**`overall_fairness_score`:** 0.0–1.0 (the dashboard shows it as a percentage)
- **0.90–1.00:** Excellent (very fair)
- **0.80–0.89:** Good (acceptable for most use cases)
- **0.70–0.79:** Fair (attention needed)
- **0.60–0.69:** Poor (action required)
- **< 0.60:** Critical (immediate action)

**Example Audit Report:**

```
Fairness Audit Report
Model: Loan Approval Model v2.1
Date: 2025-12-05 14:30 UTC
Protected Attribute: Gender
Reference Group: Male
Comparison Group: Female

Overall Fairness Score: 0.75 (Fair)

Detailed Results:

1. Demographic Parity: FAIL (0.72)
   Male approval rate: 65.2%
   Female approval rate: 46.9%
   Ratio: 0.72 (should be 0.8-1.2)
   Severity: HIGH

2. Equal Opportunity: PASS (0.88)
   Male TPR (qualified approved): 87.3%
   Female TPR (qualified approved): 76.5%
   Ratio: 0.88 (should be > 0.8)
   Severity: LOW

3. Equalized Odds: FAIL
   TPR Difference: 0.11 (HIGH)
   FPR Difference: 0.15 (HIGH)
   Both should be < 0.1
   Severity: HIGH

4. Disparate Impact: FAIL (0.72)
   Should be ≥ 0.8 (80% rule)
   Legal compliance: AT RISK
   Severity: CRITICAL

Root Cause Analysis:
- Feature "income" strongly correlated with gender (0.45)
- Training data: 65% male, 35% female (imbalanced)
- Historical bias in labels possible

Recommendations:
1. URGENT: Review for legal compliance (DI < 0.8)
2. Retrain with balanced dataset
3. Consider removing/debiasing correlated features
4. Apply fairness constraints during training
5. Adjust decision thresholds per group
6. Consult legal/compliance team

Action Items:
☐ Share with compliance team
☐ Schedule retraining
☐ Implement mitigation strategy
☐ Re-audit after changes
```

### Multi-Attribute Audits

As shown above, `sensitive_attributes` already takes a list — auditing gender, race, and age
together is one call, not a separate multi-attribute method:

```python
audit = client.fairness.audit(
    model_id="model-uuid",
    sensitive_attributes=["gender", "race", "age_group"],
    y_true=y_true,
    y_pred=y_pred,
    group_data={"gender": gender, "race": race, "age_group": age_group},
)
```

For intersectional analysis — fairness across combinations like (Male, White) vs. (Female,
Black) rather than each attribute independently — construct a combined group column yourself
(`f"{gender}_{race}"`) and pass it as its own entry in `sensitive_attributes`/`group_data`;
there's no separate endpoint that cross-tabulates attributes automatically.

### Bias Mitigation Strategies

WhiteBoxXAI's job here is diagnosis — the audit above tells you *which* metric failed and by
how much. Fixing it is a modeling decision that happens in your own training pipeline, using
whatever fairness-aware tooling you already use (or a library like
[Fairlearn](https://fairlearn.org/) or [AIF360](https://ai-fairness-360.org/)):

- **Pre-processing:** rebalance or reweight training data, and check for features correlated
  with the protected attribute (a proxy like ZIP code standing in for race) before training.
- **In-processing:** train with a fairness constraint (e.g. Fairlearn's
  `ExponentiatedGradient`) if your protected attribute is available at training time.
- **Post-processing:** adjust decision thresholds per group, and evaluate the tradeoff against
  overall performance before shipping it.

Whichever you use, re-run [`client.fairness.audit()`](#running-fairness-audits) against the
retrained or adjusted model to confirm the fix actually worked — see [Explanation Use
Cases](#explanation-use-cases) above for the same "verify against real computed data, not
before-the-fact assumptions" pattern applied to explanations.

### Continuous Fairness Monitoring

**Trend and history**, without re-running a full audit each time:

```python
client.fairness.get_bias_history(model_id="model-uuid", days=30)
client.fairness.get_metric_history(model_id="model-uuid", metric_type="demographic_parity", days=30)
client.fairness.get_latest_audit(model_id="model-uuid")
```

**Alert when fairness drops:**

```python
client.alerts.create(
    name="Fairness Score Alert",
    alert_type="fairness",
    severity="high",
    model_id="model-uuid",
    conditions=[
        {"metric_name": "fairness_score", "operator": "lt", "threshold": 0.8, "window_minutes": 10080}
    ],
    notification_channels=[{"type": "email", "target": "compliance@company.com"}],
)
```

There's no built-in scheduler for re-running an audit itself on a cadence — `audit()` needs
ground truth and group data you supply, so "run this weekly" means your own scheduled job
calling it, not a platform-side schedule. [Governance Review Boards' automated periodic
reviews](/user-guide/governance/#automated-periodic-reviews) are the closest built-in
equivalent if what you actually want is a recurring human review rather than a recurring
computation.

---

## LLM Monitoring

Token, cost, latency, safety, and RAG-quality tracking for LLM calls — the LLM equivalent of
prediction logging above. For the dashboard walkthrough (Conversations, Tokens, Costs, Safety,
RAG Quality tabs), see [Monitoring LLMs](/user-guide/#monitoring-llms) in the User Guide; this
section is the SDK surface behind it.

```python
client.llm.log_call(
    provider="openai",
    model_name="gpt-4o",
    prompt="Summarize this customer complaint...",
    completion="The customer is reporting...",
    latency_ms=842,
    prompt_tokens=120,
    completion_tokens=45,
    model_id="model-uuid",   # optional — associates the call with a registered model
)

# Cost, latency, and volume, aggregated
client.llm.usage_stats(start_time="2026-07-01T00:00:00Z", end_time="2026-07-31T23:59:59Z")
client.llm.cost_breakdown()
client.llm.trends_tokens(model_id="model-uuid", granularity="day")

# Safety — toxicity, PII, and harmful-content scoring on a piece of content
client.safety.analyze(content="...")

# RAG retrieval quality — pass ground_truth_ids to get precision/recall/MRR/NDCG computed
# server-side at log time
client.rag.log_retrieval(
    query="What's our refund policy?",
    results=[{"document_id": "doc-1", "rank": 1, "score": 0.92}],
    top_k=5,
    retrieval_method="vector",
    model_id="model-uuid",
)
client.rag.stats(model_id="model-uuid")
```

`client.llm`, `client.safety`, and `client.rag` are each their own resource with a larger
method surface than shown here (batch logging, threshold-based alerting, trend queries) — see
the [MCP Server tool reference](/integrations/mcp/#llm-monitoring) for the equivalent tool
list, which mirrors the SDK method-for-method.

## Alert System

Covered in full in the User Guide's [Managing Alerts](/user-guide/#managing-alerts) and the
[Alerts API reference](/sdk/api-reference/#alerts) — this page's [Alerts on
Metrics](#alerts-on-metrics) and [Continuous Fairness
Monitoring](#continuous-fairness-monitoring) sections above show it applied to specific
features.

## Reporting

Covered in full in [Audit & Explanation Reports](/user-guide/reports/) — report categories,
generating one from the dashboard or the API, scheduling, external sharing, and branding.

## Compliance

Covered in full in [AI Regulations](/account/ai-regulations/) (the regulatory landscape) and
[Governance Review Boards](/user-guide/governance/) (approval workflows and the decision
archive). The **Compliance** [report category](/user-guide/reports/#report-categories) is
where regulatory evidence gets packaged for an auditor.

---

*Last Updated: August 27, 2026*
