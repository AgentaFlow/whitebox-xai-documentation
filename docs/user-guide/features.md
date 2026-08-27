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
- Version control
- Model comparison
- Metadata management
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
    version="2.1.0",
    model_type="classification",
    description="Predicts customer churn risk using behavioral data",
    framework="xgboost",
    features=[
        "account_age_days",
        "total_purchases",
        "avg_purchase_value",
        "days_since_last_purchase",
        "customer_service_calls",
        "satisfaction_score"
    ],
    target="will_churn",
    baseline_metrics={
        "accuracy": 0.89,
        "precision": 0.85,
        "recall": 0.82,
        "f1_score": 0.835,
        "auc_roc": 0.93
    },
    tags=["production", "customer-retention", "high-priority"],
    metadata={
        "training_data_size": 50000,
        "training_date": "2025-11-15",
        "model_path": "s3://models/churn-v2.1.0.pkl",
        "owner": "data-science-team@company.com"
    }
)

print(f"Model registered with ID: {model.id}")
```

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

**Semantic Versioning:**
- **Major (1.0.0 → 2.0.0)** - Breaking changes, architecture change
- **Minor (1.0.0 → 1.1.0)** - New features, retrained with new data
- **Patch (1.0.0 → 1.0.1)** - Bug fixes, hyperparameter tuning

**Version Management:**
```python
# Register new version
model_v2 = client.models.register(
    name="Customer Churn Predictor",
    version="2.2.0",
    parent_version="2.1.0"  # Track lineage
)

# List all versions
versions = client.models.list_versions(name="Customer Churn Predictor")
for v in versions:
    print(f"v{v.version} - {v.status} - {v.created_at}")

# Compare versions
comparison = client.models.compare(
    model_ids=[model_v1.id, model_v2.id]
)
print(comparison.metrics_diff)
```

### Model Metadata

**Tracked Automatically:**
- Registration date
- Last updated
- Prediction count
- Last prediction timestamp
- Active alerts
- Drift status

**Custom Metadata:**
```python
client.models.update(
    model_id="model-uuid",
    metadata={
        "business_owner": "jane@company.com",
        "compliance_approved": True,
        "approval_date": "2025-12-01",
        "model_card": "https://docs.company.com/models/churn-v2",
        "risk_level": "medium",
        "data_sources": ["crm_db", "purchase_history", "support_tickets"]
    }
)
```

### Model Status Lifecycle

**Statuses:**
1. **Draft** - Registered but not deployed
2. **Staging** - Deployed to staging environment
3. **Production** - Live in production
4. **Deprecated** - Being phased out
5. **Archived** - No longer in use, data preserved

```python
# Update status
client.models.update_status(
    model_id="model-uuid",
    status="production",
    notes="Deployed to production on 2025-12-05"
)

# Archive old model
client.models.archive(
    model_id="old-model-uuid",
    reason="Replaced by v2.2.0"
)
```

### Model Organization

**Tags:**
```python
# Add tags
client.models.add_tags(model_id, ["production", "high-priority"])

# Search by tags
prod_models = client.models.list(tags=["production"])
```

**Folders (UI only):**
- Organize models into folders
- By team: "Marketing", "Risk", "Product"
- By use case: "Fraud Detection", "Recommendation", "Forecasting"

**Favorites:**
- Star frequently accessed models
- Quick access from dashboard

### Model Comparison

Compare multiple models side-by-side:

**Via UI:**
1. Go to **Models** page
2. Select 2-4 models (checkboxes)
3. Click **Compare** button
4. View comparison tabs:
   - **Overview** - Side-by-side info
   - **Metrics** - Performance comparison
   - **Trends** - Metrics over time
   - **Configuration** - Settings differences

**Via SDK:**
```python
comparison = client.models.compare(
    model_ids=["model-1-uuid", "model-2-uuid"],
    metrics=["accuracy", "precision", "recall"],
    date_range="last_30_days"
)

print(f"Accuracy: {comparison.metrics['accuracy']}")
# Output: {'model-1': 0.89, 'model-2': 0.91}

print(f"Winner: {comparison.best_model}")
# Output: model-2 (higher accuracy)
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
- Input features
- Model output (prediction, probability)
- Timestamp
- Metadata (optional)
- Ground truth label (when available)

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

print(f"Logged: {response.prediction_id}")
```

#### 2. Decorator Pattern

Automatically log by decorating your prediction function:

```python
from whiteboxxai.decorators import log_predictions

@log_predictions(
    client=client,
    model_id="model-uuid",
    feature_names=["age", "income", "credit_score", "debt_ratio"]
)
def predict_loan_approval(age, income, credit_score, debt_ratio):
    """Make prediction - logging happens automatically"""
    features = np.array([[age, income, credit_score, debt_ratio]])
    prediction = model.predict(features)[0]
    probability = model.predict_proba(features)[0][1]

    return {
        "prediction": int(prediction),
        "probability": float(probability)
    }

# Use normally - automatic logging
result = predict_loan_approval(35, 75000, 720, 0.35)
```

#### 3. Batch Logging

Log multiple predictions at once (more efficient):

```python
predictions = []

for customer in customers:
    features = extract_features(customer)
    pred = model.predict([features])
    prob = model.predict_proba([features])[0][1]

    predictions.append({
        "inputs": features,
        "output": {
            "prediction": int(pred[0]),
            "probability": float(prob)
        },
        "metadata": {"customer_id": customer.id}
    })

# Log batch (up to 1000 per call)
response = client.predictions.log_batch(
    model_id="model-uuid",
    predictions=predictions
)

print(f"Logged {response.count} predictions")
```

#### 4. Framework Integration

**Scikit-learn:**
```python
from whiteboxxai.integrations.sklearn import wrap_model

monitored_model = wrap_model(
    model=my_sklearn_model,
    client=client,
    model_id="model-uuid",
    feature_names=["age", "income", "credit_score"]
)

# Use like normal - logging automatic
predictions = monitored_model.predict(X_test)
probabilities = monitored_model.predict_proba(X_test)
```

**PyTorch:**
```python
from whiteboxxai.integrations.pytorch import MonitoredModel

class MyModel(MonitoredModel):
    def __init__(self, model, client, model_id):
        super().__init__(model, client, model_id)

    def forward(self, x):
        output = self.model(x)
        self.log_prediction(x, output)  # Auto-logs
        return output
```

### Sampling Strategies

**When to Sample:**
- High-volume models (>10k predictions/day)
- Cost optimization
- Reduce storage

**Sampling Methods:**

**1. Uniform Random Sampling:**
```python
client.models.update(
    model_id="model-uuid",
    sampling_rate=0.1  # Log 10% randomly
)
```

**2. Stratified Sampling:**
```python
# Log all high-confidence predictions, sample low-confidence
if probability > 0.9 or probability < 0.1:
    sample_rate = 1.0  # Log all
else:
    sample_rate = 0.05  # Log 5%

if random.random() < sample_rate:
    client.predictions.log(...)
```

**3. Time-based Sampling:**
```python
# Log more during business hours
import datetime
hour = datetime.datetime.now().hour
if 9 <= hour <= 17:  # Business hours
    sample_rate = 0.2
else:
    sample_rate = 0.05
```

**Recommendations:**
- **< 1k/day:** 100% (log all)
- **1k-10k/day:** 10-50%
- **10k-100k/day:** 1-10%
- **> 100k/day:** 0.1-1%

### Ground Truth Labels

Provide actual outcomes to calculate accuracy:

**Update Later:**
```python
# When ground truth becomes available
client.predictions.update(
    prediction_id="pred-uuid",
    actual_label=1,
    actual_timestamp="2025-12-10T10:30:00Z"
)
```

**Batch Update:**
```python
updates = [
    {"prediction_id": "pred-1", "actual_label": 0},
    {"prediction_id": "pred-2", "actual_label": 1},
    {"prediction_id": "pred-3", "actual_label": 0},
]

client.predictions.update_batch(updates)
```

**Webhook Integration:**
```python
# Configure webhook to receive labels automatically
client.models.configure_webhook(
    model_id="model-uuid",
    url="https://yourapp.com/webhook/ground-truth",
    events=["label.available"]
)
```

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

Define business-specific metrics:

```python
# Log custom metric
client.metrics.log_custom(
    model_id="model-uuid",
    metric_name="revenue_impact",
    metric_value=12500.50,
    metric_type="currency",
    timestamp=datetime.now()
)

# View custom metrics
metrics = client.metrics.get_custom(
    model_id="model-uuid",
    metric_name="revenue_impact",
    date_range="last_30_days"
)
```

**Common Custom Metrics:**
- Revenue impact
- Cost savings
- Customer satisfaction
- Processing time
- Business conversion rate

### Metric Aggregations

**Time Windows:**
- Real-time (last 5 minutes)
- Hourly
- Daily
- Weekly
- Monthly
- Custom range

**Aggregation Functions:**
- Average (mean)
- Median
- Min/Max
- Percentiles (p50, p95, p99)
- Sum
- Count

```python
# Get metrics with aggregation
metrics = client.metrics.get(
    model_id="model-uuid",
    metric_names=["accuracy", "latency"],
    aggregation="mean",
    granularity="daily",
    date_range="last_30_days"
)
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
- **Wasserstein Distance** - Earth mover's distance
- **Population Stability Index (PSI)** - Industry standard

**Categorical Features:**
- **Chi-squared Test** - Compares frequency distributions
- **Jensen-Shannon Divergence** - Symmetric KL divergence

**Implementation:**
```python
# Configure data drift detection
client.models.configure_drift(
    model_id="model-uuid",
    drift_config={
        "enabled": True,
        "detection_method": "ks_test",
        "threshold": 0.1,  # p-value threshold
        "reference_period": "training",  # or "7d", "30d"
        "detection_frequency": "daily",
        "features": ["income", "age", "credit_score"]  # or "all"
    }
)
```

**Drift Score Interpretation:**
- **0.0 - 0.05:** No drift (very similar)
- **0.05 - 0.1:** Low drift (minor changes)
- **0.1 - 0.2:** Medium drift (investigate)
- **0.2+:** High drift (action required)

### Concept Drift

**Definition:**
Changes in the relationship between features and target.

**Example:**
Credit score of 650 used to have 20% default rate, now has 10% default rate.

**Detection Methods:**

**Error-based:**
- Track prediction accuracy over time
- Compare recent vs historical performance
- Alert on performance degradation

**Distribution-based:**
- Monitor joint distribution P(X, Y)
- Use multivariate drift tests
- Compare conditional distributions P(Y|X)

**Implementation:**
```python
# Concept drift detection (automatic with ground truth)
client.models.configure_drift(
    model_id="model-uuid",
    concept_drift={
        "enabled": True,
        "method": "performance_degradation",
        "baseline_window": "30d",
        "detection_window": "7d",
        "threshold": 0.05  # 5% accuracy drop
    }
)
```

### Prediction Drift

**Definition:**
Changes in model output distribution.

**Example:**
Fraud rate predictions increased from 5% to 15% of traffic.

**Use Cases:**
- Detect model behavior changes
- Identify bias drift
- Monitor deployed model vs champion

```python
# Prediction drift monitoring
client.drift.monitor_predictions(
    model_id="model-uuid",
    config={
        "track_distribution": True,
        "alert_on_shift": True,
        "threshold": 0.15  # 15% shift in distribution
    }
)
```

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
# Get drift details
drift = client.drift.get_latest(model_id="model-uuid")

# View drifted features
for feature in drift.drifted_features:
    print(f"{feature.name}: {feature.score} ({feature.severity})")

# Compare distributions
comparison = client.drift.compare_distributions(
    model_id="model-uuid",
    feature="transaction_amount",
    reference="training",
    current="last_7d"
)

# View visualization
comparison.plot()  # Shows overlaid histograms
```

**3. Action:**
- **Low drift:** Monitor, no action yet
- **Medium drift:** Plan retraining within 1-2 weeks
- **High drift:** Immediate retraining or model rollback

**4. Retraining:**
```python
# After retraining and deploying new version
client.models.register(
    name="Fraud Detection",
    version="2.3.0",  # New version
    parent_version="2.2.0",
    baseline_metrics={...},  # New baselines
    metadata={"retrained_due_to": "data_drift"}
)

# Update drift baseline
client.models.update_drift_baseline(
    model_id="new-model-uuid",
    baseline_data="recent"  # Use recent production data
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
- Feature Importance
- Partial Dependence Plots
- Individual Conditional Expectation (ICE)

### SHAP Explanations

**What is SHAP?**

Based on game theory (Shapley values), SHAP provides:
- Theoretically sound explanations
- Local and global interpretability
- Consistent feature attributions
- Additive property

**How it Works:**

For each prediction:
1. Calculate contribution of each feature
2. Base value (average prediction) + feature contributions = final prediction
3. Positive contribution = pushes prediction higher
4. Negative contribution = pushes prediction lower

**Generate SHAP Explanation:**

```python
# Via SDK
explanation = client.explanations.generate(
    prediction_id="pred-uuid",
    method="shap",
    config={
        "background_samples": 100,  # For TreeSHAP
        "approximate": False
    }
)

# Access results
print(f"Base value: {explanation.base_value}")
print(f"Prediction: {explanation.prediction}")

for feature, value in explanation.feature_contributions.items():
    print(f"{feature}: {value:+.3f}")
```

**Visualizations:**

**1. Waterfall Chart:**
Shows cumulative feature contributions

```
Base Value: 0.20
+ income (high): +0.15
+ credit_score (excellent): +0.25
+ debt_ratio (low): +0.10
- age (young): -0.05
= Final Prediction: 0.65
```

**2. Force Plot:**
Visual representation of forces pushing prediction up/down

```python
explanation.plot_force()
```

**3. Summary Plot (Global):**
Feature importance across all predictions

```python
client.explanations.plot_summary(
    model_id="model-uuid",
    num_predictions=1000
)
```

**4. Dependence Plot:**
Shows relationship between feature value and SHAP value

```python
client.explanations.plot_dependence(
    model_id="model-uuid",
    feature="credit_score"
)
```

### LIME Explanations

**What is LIME?**

Creates a simple, interpretable model around a specific prediction:
1. Generate perturbations around the instance
2. Get model predictions for perturbations
3. Train a simple linear model on these
4. Use linear model coefficients as explanations

**When to Use LIME:**
- Faster than SHAP for complex models
- Good for high-dimensional data
- Need quick explanations
- Local understanding sufficient

**Generate LIME Explanation:**

```python
explanation = client.explanations.generate(
    prediction_id="pred-uuid",
    method="lime",
    config={
        "num_samples": 5000,
        "num_features": 10  # Top 10 features
    }
)

# Feature weights
for feature, weight in explanation.feature_weights.items():
    direction = "increases" if weight > 0 else "decreases"
    print(f"{feature}: {abs(weight):.3f} {direction} prediction")
```

**Output Example:**
```
Prediction: Approved (probability: 0.85)

Top Contributing Features:
+ credit_score (720): +0.35 (high score increases approval)
+ income ($75k): +0.28 (sufficient income increases approval)
+ employment_years (8): +0.15 (stable employment increases approval)
- debt_ratio (0.35): -0.12 (moderate debt decreases approval)
- recent_inquiries (3): -0.08 (recent credit checks decrease approval)
```

### Feature Importance

**Global Model Understanding:**

```python
# Get global feature importance
importance = client.explanations.get_feature_importance(
    model_id="model-uuid",
    method="shap",  # or "permutation", "gain"
    num_predictions=1000
)

# Top features
for feature, score in importance.items():
    print(f"{feature}: {score:.3f}")
```

**Visualization:**
```python
importance.plot_bar()  # Horizontal bar chart
importance.plot_beeswarm()  # SHAP beeswarm plot
```

### Explanation Use Cases

**1. Debugging:**
```python
# Find predictions where model relies heavily on unexpected features
explanations = client.explanations.get(
    model_id="model-uuid",
    filter={
        "top_feature": "zip_code",  # Shouldn't be most important
        "limit": 100
    }
)

# Investigate why zip_code is driving decisions
```

**2. Contestation:**
```python
# Customer disputes loan denial - provide explanation
explanation = client.explanations.generate(
    prediction_id=denial_prediction_id,
    method="shap"
)

# Generate customer-friendly report
report = client.explanations.export_report(
    explanation_id=explanation.id,
    format="pdf",
    template="customer_friendly"
)
```

**3. Compliance:**
```python
# Generate explanation for audit
explanation = client.explanations.generate(
    prediction_id="pred-uuid",
    method="shap",
    metadata={
        "audit_request_id": "AUD-2025-0123",
        "requested_by": "compliance@company.com",
        "purpose": "regulatory_audit"
    }
)

# Explanation automatically logged for audit trail
```

**4. Model Improvement:**
```python
# Identify features with low importance - candidates for removal
global_importance = client.explanations.get_feature_importance(
    model_id="model-uuid"
)

low_importance = [
    feature for feature, score in global_importance.items()
    if score < 0.01
]

print(f"Consider removing: {low_importance}")
```

### Explanation Configuration

```python
# Set default explanation method per model
client.models.update(
    model_id="model-uuid",
    explanation_config={
        "default_method": "shap",
        "auto_generate": False,  # Generate on-demand only
        "cache_duration": "30d",
        "background_samples": 100
    }
)
```

### Explanation API

**Batch Generation:**
```python
# Generate explanations for multiple predictions
results = client.explanations.generate_batch(
    prediction_ids=["pred-1", "pred-2", "pred-3"],
    method="shap"
)

for result in results:
    print(f"{result.prediction_id}: {result.status}")
```

**Async Generation:**
```python
# For slow explanations, generate asynchronously
task = client.explanations.generate_async(
    prediction_id="pred-uuid",
    method="shap"
)

# Check status later
status = client.explanations.get_task_status(task.id)
if status.completed:
    explanation = status.result
```

---

*Continued in next message due to length...*

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

```python
# Run comprehensive fairness audit
audit = client.bias.run_audit(
    model_id="model-uuid",
    protected_attribute="gender",
    reference_group="male",
    metrics=[
        "demographic_parity",
        "equal_opportunity",
        "equalized_odds",
        "disparate_impact"
    ],
    date_range="last_30_days"
)

# Overall fairness score
print(f"Fairness Score: {audit.overall_score}/100")

# Per-metric results
for metric_name, result in audit.metrics.items():
    print(f"{metric_name}: {result.value:.3f} ({result.status})")

# Recommendations
for rec in audit.recommendations:
    print(f"- {rec}")
```

### Understanding Audit Results

**Overall Fairness Score:** 0-100
- **90-100:** Excellent (very fair)
- **80-89:** Good (acceptable for most use cases)
- **70-79:** Fair (attention needed)
- **60-69:** Poor (action required)
- **< 60:** Critical (immediate action)

**Example Audit Report:**

```
Fairness Audit Report
Model: Loan Approval Model v2.1
Date: 2025-12-05 14:30 UTC
Protected Attribute: Gender
Reference Group: Male
Comparison Group: Female

Overall Fairness Score: 75/100 (Fair)

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

Audit multiple protected attributes:

```python
# Audit gender, race, and age simultaneously
audit = client.bias.run_multi_audit(
    model_id="model-uuid",
    protected_attributes=["gender", "race", "age_group"],
    reference_groups={
        "gender": "male",
        "race": "white",
        "age_group": "30-50"
    }
)

# Intersectional analysis
intersectional = client.bias.analyze_intersectional(
    model_id="model-uuid",
    attributes=["gender", "race"],
    # Analyzes: (Male, White), (Male, Black), (Female, White), (Female, Black)
)

for group, fairness_score in intersectional.items():
    print(f"{group}: {fairness_score}")
```

### Bias Mitigation Strategies

**1. Pre-processing (Training Data):**

```python
# Rebalance training data
from sklearn.utils import resample

# Upsample minority group
female_upsampled = resample(
    female_data,
    n_samples=len(male_data),
    random_state=42
)

balanced_data = pd.concat([male_data, female_upsampled])

# Remove biased features
features_to_remove = client.bias.identify_proxy_features(
    model_id="model-uuid",
    protected_attribute="gender",
    correlation_threshold=0.3
)

print(f"Correlated features: {features_to_remove}")
# ['zip_code', 'job_title', 'hobby']
```

**2. In-processing (During Training):**

```python
from whiteboxxai.fairness import FairClassifier

# Train with fairness constraints
model = FairClassifier(
    base_model=LogisticRegression(),
    protected_attribute="gender",
    fairness_constraint="demographic_parity",
    epsilon=0.05  # Allow 5% deviation
)

model.fit(X_train, y_train, sensitive_features=A_train)
```

**3. Post-processing (Adjust Predictions):**

```python
# Adjust decision thresholds per group
thresholds = client.bias.optimize_thresholds(
    model_id="model-uuid",
    protected_attribute="gender",
    fairness_metric="equal_opportunity",
    optimize_for="f1_score"  # Maintain performance
)

print(f"Threshold for male: {thresholds['male']}")
print(f"Threshold for female: {thresholds['female']}")

# Apply in production
def fair_predict(features, gender):
    probability = model.predict_proba(features)[1]
    threshold = thresholds[gender]
    return 1 if probability >= threshold else 0
```

### Continuous Fairness Monitoring

**Set up alerts:**

```python
client.alerts.create(
    name="Fairness Score Alert",
    alert_type="fairness",
    severity="high",
    model_id="model-uuid",
    conditions=[
        {"metric_name": "fairness_score", "operator": "lt", "threshold": 80, "window_minutes": 10080}
    ],
    notification_channels=[{"type": "email", "target": "compliance@company.com"}],
)
```

**Schedule regular audits:**

```python
client.bias.schedule_audit(
    model_id="model-uuid",
    frequency="weekly",  # or "daily", "monthly"
    protected_attributes=["gender", "race"],
    recipients=["ml-team@company.com", "compliance@company.com"]
)
```

---

*Due to length, continuing with remaining features...*

## LLM Monitoring

[Content for LLM Monitoring, Alert System, Reporting, and Compliance features would follow the same detailed format as above, covering all aspects of each feature]

---

*Last Updated: December 5, 2025*
*Version: 1.0*
