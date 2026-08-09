# WhiteBoxXAI User Guide

Welcome to WhiteBoxXAI! This comprehensive guide will help you understand and use all features of the platform.

---

## Table of Contents

1. [Introduction](#introduction)
2. [Getting Started](#getting-started)
3. [Dashboard Overview](#dashboard-overview)
4. [Managing Models](#managing-models)
5. [Monitoring Predictions](#monitoring-predictions)
6. [Understanding Explanations](#understanding-explanations)
7. [Detecting Drift](#detecting-drift)
8. [Auditing for Bias](#auditing-for-bias)
9. [Monitoring LLMs](#monitoring-llms)
10. [Managing Alerts](#managing-alerts)
11. [Evidence & Reports](#evidence--reports)
12. [User Settings](#user-settings)

**Governance & compliance features have their own pages:**

- [Trust Score](/user-guide/trust-score/) — the 0–100 index over fairness, drift, and explainability
- [AI Risk Register](/user-guide/risk-register/) — structured risk inventory with owners and workflow
- [Governance Review Boards](/user-guide/governance/) — multi-party approval and decision archive

---

## Introduction

### What is WhiteBoxXAI?

WhiteBoxXAI is an AI Observability & Explainability platform that helps you:

- **Monitor** your ML models in production
- **Explain** how your models make predictions
- **Detect** when your models drift or degrade
- **Audit** your models for bias and fairness
- **Comply** with AI regulations and standards

### Who Should Use WhiteBoxXAI?

- **Data Scientists** - Monitor model performance and get insights
- **ML Engineers** - Debug models and track drift
- **Compliance Officers** - Audit models for regulatory compliance
- **Product Managers** - Understand model behavior and impact
- **Executives** - Get high-level insights into AI systems

### Key Features

✅ Real-time model monitoring
✅ SHAP and LIME explanations
✅ Automatic drift detection
✅ Bias and fairness auditing
✅ LLM observability
✅ [Trust Score](/user-guide/trust-score/) — one 0–100 index per model
✅ [AI Risk Register](/user-guide/risk-register/) with owners, workflow, and audit trail
✅ [Governance review boards](/user-guide/governance/) with an immutable decision archive
✅ Compliance evidence exports
✅ Customizable alerts

---

## Getting Started

### Creating Your Account

1. Go to [app.whiteboxxai.com](https://app.whiteboxxai.com)
2. Click **Sign Up**
3. Enter your email and create a password
4. Verify your email address
5. Log in to your account

### First Login

After logging in, you'll see:

- **Dashboard** - Overview of your models and metrics
- **Navigation Menu** - Access to all features
- **Quick Actions** - Press `Cmd/Ctrl + K` for quick navigation (if enabled)

### Setting Up Your Profile

1. Click your avatar in the top-right corner
2. Select **Profile**
3. Update your information:
   - Full name
   - Email address
   - Avatar image
   - Notification preferences

---

## Dashboard Overview

### Operations Dashboard

The default dashboard gives you a high-level operational view of your AI systems:

**Key Metrics Cards:**
- **Total Models** - Number of registered models
- **Active Models** - Currently monitored models
- **Total Predictions** - Predictions logged today
- **Active Alerts** - Current alerts requiring attention

**Charts & Visualizations:**
- Prediction volume over time
- Model performance trends
- Alert history
- Drift detection summary

**Recent Activity:**
- Latest predictions
- Recent alerts
- New drift detections
- System notifications

!!! note "A new account starts empty — on purpose"
    Until you register a model and log predictions, these cards and charts show empty states
    with a quick-start snippet rather than numbers. WhiteBoxXAI never displays placeholder or
    sample figures in your own workspace: every value you see is computed from your data.
    If you want to explore a populated dashboard, [seed demo
    data](/get-started/getting-started/#step-4-first-run-experience) or browse the
    read-only [Demo plan](/get-started/plans/#demo).

### Executive Dashboard

A board-ready summary aimed at stakeholders who don't work in the platform day to day:
portfolio [Trust Score](/user-guide/trust-score/), lowest-scoring models, risk posture, and ROI
framing. Find it at **Overview → Executive Dashboard**.

### Navigation

The sidebar is organized into five groups:

**Overview**

- **Operations Dashboard** - Day-to-day operational view
- **Executive Dashboard** - Portfolio Trust Score and risk posture
- **Models** - Manage your models

**Assurance**

- **Explainability** - SHAP and LIME explanations
- **Fairness** - Bias and fairness audits
- **Drift Detection** - Data and concept drift

**Observability**

- **Alerts** - Active alerts and alert rules
- **Performance** - Real-time metrics and trends
- **LLM Monitoring** - Language model tracking, cost, and safety
- **Multi-Agent Workflows** - Agent workflow tracing

**Governance & Evidence**

- **Compliance** - Regulatory framework coverage, including ISO 42001
- **Risk Register** - [Structured AI risk inventory](/user-guide/risk-register/)
- **Evidence & Reports** - Exports and scheduled reports
- **Review Boards** - [Governance review boards](/user-guide/governance/)
- **My Requests** - Review requests you submitted or need to vote on
- **Decisions Archive** - Searchable, immutable record of finalized decisions

**Configuration**

- **Documentation** - This site
- **Settings** - Organization and account settings

**Quick Actions (Cmd/Ctrl + K):**
Type to search for:
- Pages (e.g., "models", "alerts")
- Actions (e.g., "register model", "create alert")
- Settings (e.g., "profile", "notifications")

---

## Managing Models

### Registering a New Model

**Step 1: Navigate to Models**
1. Click **Models** in the sidebar
2. Click **Register Model** button

**Step 2: Basic Information**
- **Name** - Descriptive name (e.g., "Fraud Detection v2")
- **Description** - Purpose and use case
- **Type** - Classification, Regression, Clustering, or LLM
- **Framework** - scikit-learn, PyTorch, TensorFlow, etc.
- **Version** - Model version (e.g., "1.0.0")
- **Tags** - Labels for organization (e.g., "production", "finance")

**Step 3: Model Metadata**
- **Target Variable** - What the model predicts
- **Features** - Input features (comma-separated)
- **Training Data Size** - Number of training samples
- **Training Date** - When the model was trained

**Step 4: Baseline Metrics**
- **Accuracy** - Overall accuracy on test set
- **Precision** - Positive prediction accuracy
- **Recall** - True positive rate
- **F1 Score** - Harmonic mean of precision and recall

**Step 5: Monitoring Configuration**
- **Sampling Rate** - Percentage of predictions to log (0.0-1.0)
- **Drift Threshold** - Sensitivity for drift detection
- **Enable Monitoring** - Start monitoring immediately

**Step 6: Review & Submit**
- Review all information
- Click **Register Model**
- You'll receive a model ID for SDK integration

### Viewing Model Details

1. Go to **Models** page
2. Click on any model name
3. View tabs:
   - **Overview** - Basic information and stats
   - **Predictions** - Recent predictions
   - **Metrics** - Performance over time
   - **Drift** - Drift detection results
   - **Explanations** - Generated explanations
   - **Configuration** - Model settings

### Editing a Model

1. Open model details
2. Click **Edit Model** button
3. Update information in tabs:
   - Basic Info
   - Monitoring settings
   - Alert configuration
4. Click **Save Changes**

### Comparing Models

1. Go to **Models** page
2. Click **Compare Models** button
3. Select 2-4 models to compare
4. View comparison tabs:
   - **Overview** - Side-by-side comparison
   - **Performance Trends** - Metrics over time
   - **Metrics Comparison** - Current metrics
   - **Configuration** - Settings differences

### Archiving a Model

To archive a model no longer in use:

1. Open model details
2. Go to **Advanced** tab
3. Click **Archive Model**
4. Confirm the action

**Note:** Archived models retain all data but stop monitoring.

---

## Monitoring Predictions

### Logging Predictions

**Using the SDK:**
```python
from whiteboxxai import WhiteBoxXAI

client = WhiteBoxXAI(api_key="your-api-key")

# Log a single prediction
client.predictions.log(
    model_id="model-uuid",
    input_data={"feature1": 1.5, "feature2": "value"},
    output_data={"prediction": 0, "probability": 0.92}
)
```

**Using the API:**
```bash
curl -X POST https://api.whiteboxxai.com/api/v1/predictions/log \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "model_id": "model-uuid",
    "inputs": {"feature1": 1.5},
    "output": {"prediction": 0}
  }'
```

### Viewing Predictions

1. Go to model details page
2. Click **Predictions** tab
3. View prediction log table with:
   - Prediction ID
   - Timestamp
   - Input values
   - Output values
   - Confidence score

**Filters:**
- Date range
- Prediction value
- Confidence threshold
- Search by ID

### Analyzing Prediction Patterns

**Prediction Volume:**
- View chart showing predictions over time
- Identify traffic patterns
- Detect anomalies

**Prediction Distribution:**
- See distribution of predicted values
- Compare to training distribution
- Spot unusual patterns

**Latency Analysis:**
- Track prediction response times
- Identify performance issues
- Set latency alerts

---

## Understanding Explanations

### What are Model Explanations?

Explanations help you understand **why** your model made a specific prediction by showing:
- Which features contributed most
- How each feature impacted the prediction
- The direction of influence (positive/negative)

### Generating an Explanation

**Method 1: From Prediction Log**
1. Go to model's **Predictions** tab
2. Click on a prediction
3. Click **Generate Explanation**
4. Select method (SHAP or LIME)
5. Wait for processing (~5-30 seconds)

**Method 2: From Explanations Page**
1. Go to **Explanations** in sidebar
2. Click **New Explanation**
3. Select model and prediction ID
4. Choose explanation method
5. Click **Generate**

### Understanding SHAP Explanations

**SHAP (SHapley Additive exPlanations)** shows how each feature contributes to the prediction.

**Waterfall Chart:**
- Shows cumulative feature contributions
- Base value (average prediction) → Final prediction
- Red bars = push prediction higher
- Blue bars = push prediction lower

**Force Plot:**
- Visualizes feature forces
- Features pushing toward/away from threshold
- Magnitude indicates strength of impact

**Example Interpretation:**
```
Base Value: 0.2 (20% fraud probability)
+ Amount (high): +0.45
+ Time (unusual): +0.15
- Merchant (trusted): -0.12
= Final Prediction: 0.68 (68% fraud probability)
```

### Understanding LIME Explanations

**LIME (Local Interpretable Model-agnostic Explanations)** creates a simple model around the prediction.

**Feature Weights:**
- Positive weights increase prediction
- Negative weights decrease prediction
- Absolute value = importance

**Example:**
```
Prediction: Approved (0.85 probability)

Top Features:
+ Credit Score: +0.35 (high score helps)
+ Income: +0.28 (sufficient income)
- Debt Ratio: -0.12 (moderate debt)
```

### Global Feature Importance

View overall feature importance across all predictions:

1. Go to **Explanations** page
2. Click **Global Importance** tab
3. See ranked features with importance scores

**Use Cases:**
- Understand which features matter most
- Identify unused or redundant features
- Validate model behavior
- Document model decisions for compliance

---

## Detecting Drift

### What is Model Drift?

**Data Drift:** Input feature distributions change
**Concept Drift:** Relationship between features and target changes

**Why it Matters:**
- Models trained on old data may not work on new data
- Performance degrades over time
- Predictions become unreliable

### Drift Detection Dashboard

Access: **Drift Detection** in sidebar

**Overview Cards:**
- Models with Drift
- High Severity Detections
- Recent Drift Events
- Average Drift Score

**Drift Timeline:**
- Chronological drift events
- Severity indicators (Low, Medium, High)
- Affected features

### Understanding Drift Reports

**Data Drift Report Shows:**
- Overall drift score (0-1, higher = more drift)
- Per-feature drift scores
- Statistical test results (KS test, Chi-squared)
- Distribution comparisons
- Severity classification

**Example:**
```
Model: Fraud Detection v2
Detection Date: 2025-12-05
Overall Score: 0.15 (Medium)

Drifted Features:
- transaction_amount: 0.22 (High) - values shifted higher
- merchant_category: 0.18 (Medium) - distribution changed
- time_of_day: 0.08 (Low) - slight variation
```

### Responding to Drift

**When drift is detected:**

1. **Review the Report**
   - Which features drifted?
   - How severe is the drift?
   - When did it start?

2. **Investigate Root Cause**
   - Business changes (new products, markets)
   - Data quality issues
   - Seasonal patterns
   - External events

3. **Take Action**
   - **Low Severity:** Monitor closely
   - **Medium Severity:** Plan retraining
   - **High Severity:** Retrain model immediately or revert to previous version

4. **Update the Model**
   - Retrain with recent data
   - Register new version in WhiteBoxXAI
   - Update baseline metrics

### Configuring Drift Detection

1. Go to model details
2. Click **Monitoring** tab
3. Configure:
   - **Detection Frequency** - How often to check (hourly, daily)
   - **Threshold** - Sensitivity (0.05 = very sensitive, 0.2 = less sensitive)
   - **Reference Period** - Compare against last 7/30/90 days
   - **Alert Notifications** - Who to notify when drift detected

---

## Auditing for Bias

### What is Bias in ML?

**Bias** occurs when a model performs differently across demographic groups, potentially leading to unfair outcomes.

**Protected Attributes:** Characteristics that should not influence predictions:
- Race/Ethnicity
- Gender
- Age
- Religion
- Disability status

### Running a Fairness Audit

1. Go to **Bias & Fairness** page
2. Click **Run Audit**
3. Configure audit:
   - **Model** - Select model to audit
   - **Protected Attributes** - Select attributes (e.g., gender, age)
   - **Reference Group** - Baseline group for comparison
   - **Fairness Metrics** - Select metrics to calculate
   - **Time Period** - Date range for analysis

4. Click **Run Audit**
5. Wait for processing (~1-3 minutes)

### Understanding Fairness Metrics

**Demographic Parity:**
- Groups should have similar positive prediction rates
- Example: Approval rates should be similar across genders
- **Pass:** Ratio between 0.8 and 1.2

**Equal Opportunity:**
- True positive rates should be similar across groups
- Example: Qualified applicants approved at similar rates
- **Pass:** Difference < 0.1

**Equalized Odds:**
- Both true positive and false positive rates should be equal
- Strictest fairness criterion
- **Pass:** Both differences < 0.1

### Interpreting Audit Results

**Overall Fairness Score:** 0-100
- **90-100:** Excellent (very fair)
- **80-89:** Good (acceptable)
- **70-79:** Fair (needs attention)
- **<70:** Poor (immediate action required)

**Example Report:**
```
Model: Loan Approval Model
Audit Date: 2025-12-05
Overall Score: 75 (Fair)

Protected Attribute: Gender
Reference Group: Male

Demographic Parity: FAIL (0.72)
- Male approval rate: 65%
- Female approval rate: 47%
- Ratio: 0.72 (should be 0.8-1.2)

Equal Opportunity: PASS (0.88)
- Similar approval rates for qualified applicants

Recommendations:
1. Review training data for gender balance
2. Consider reweighting underrepresented groups
3. Audit feature selection for gender correlation
```

### Taking Action on Bias

**If bias is detected:**

1. **Document Finding**
   - Save audit report
   - Share with stakeholders
   - Create improvement plan

2. **Investigate Causes**
   - Imbalanced training data?
   - Correlated features (proxies)?
   - Biased labels?

3. **Mitigation Strategies**
   - **Pre-processing:** Balance training data, remove biased features
   - **In-processing:** Use fairness constraints during training
   - **Post-processing:** Adjust decision thresholds per group

4. **Re-audit**
   - Train new model with fixes
   - Run fairness audit again
   - Compare improvements

---

## Monitoring LLMs

### LLM Monitoring Dashboard

Access: **LLM Monitoring** in sidebar

**Overview Metrics:**
- Total Completions
- Total Cost
- Average Latency
- High-Risk Responses (toxicity)

**Tabs:**
1. **Overview** - Summary statistics
2. **Conversations** - Request/response history
3. **Tokens** - Token usage analysis
4. **Costs** - Spending tracking
5. **Safety** - Toxicity detection
6. **RAG Quality** - Retrieval metrics (if using RAG)

### Viewing Conversations

1. Go to **Conversations** tab
2. Browse conversation history with:
   - Timestamp
   - Model name (GPT-4, Claude, etc.)
   - Prompt preview
   - Response preview
   - Tokens used
   - Cost

3. Click any conversation to view:
   - Full prompt
   - Full completion
   - Metadata (temperature, max tokens)
   - Toxicity scores
   - Performance metrics

### Token Usage Analysis

**Token Breakdown:**
- Input tokens (prompt)
- Output tokens (completion)
- Total tokens
- Cost per request

**Visualizations:**
- Token usage over time
- By model comparison
- By use case distribution

**Optimization Tips:**
- Reduce prompt length where possible
- Use appropriate max_tokens limits
- Choose cost-effective models for simple tasks
- Cache common responses

### Cost Tracking

Monitor LLM spending:

**Cost Metrics:**
- Total spend (today/week/month)
- Cost per model
- Cost per use case
- Cost trends over time

**Budgets & Alerts:**
1. Go to **Settings** → **LLM Budget**
2. Set monthly budget
3. Configure alerts at 50%, 75%, 90%
4. Receive notifications when thresholds reached

### Safety Monitoring

**Toxicity Detection:**

Automatically scans completions for:
- Toxicity
- Severe toxicity
- Obscenity
- Threats
- Insults
- Identity attacks

**Safety Score:** 0-1 (lower is better)
- **0.0-0.3:** Safe
- **0.3-0.6:** Moderate concern
- **0.6-1.0:** High risk

**PII Detection:**
- Automatically detects personal information
- Email addresses, phone numbers
- Credit cards, SSNs
- Names, addresses

**Actions:**
1. Review high-risk completions
2. Update prompts to reduce toxic outputs
3. Implement output filtering
4. Add safety instructions to system prompts

### RAG Quality Metrics

If using Retrieval-Augmented Generation:

**Retrieval Quality:**
- **Precision@K** - Relevance of retrieved documents
- **Recall@K** - Coverage of relevant documents
- **MRR** - Mean Reciprocal Rank
- **NDCG** - Ranking quality

**Answer Quality:**
- **Faithfulness** - Answer supported by context
- **Relevance** - Answer addresses the question
- **Context Utilization** - How much context was used

**Monitoring:**
1. Go to **RAG Quality** tab
2. View metrics over time
3. Identify retrieval issues
4. Optimize embedding strategy or retrieval parameters

---

## Managing Alerts

Alerts are managed from the dashboard at **Observability → Alerts**, and delivered through
your configured notification channels.

!!! warning "Alerts are dashboard-managed today"
    There is no public REST API for alerts yet — `/api/v1/alerts` is not available, and
    requests to it return `404`. The SDK exposes `client.alerts.*` and
    `ModelMonitor.create_alert_rule()` / `get_active_alerts()`, but those call that endpoint
    and will fail until it ships. Create and manage alert rules in the dashboard for now.

    Drift and bias thresholds *are* configurable via the API, and a `high` or `critical`
    result there will auto-draft an entry in your [AI Risk Register](/user-guide/risk-register/) — see
    [Automatic risk drafting](/user-guide/risk-register/#automatic-risk-drafting).

### Alert Dashboard

Access: **Observability → Alerts** in the sidebar

**Overview:**
- Active Alerts
- Acknowledged Alerts
- Resolved Alerts
- False Positives

**Alert Types:**
- 🔴 **Critical** - Immediate attention required
- 🟠 **High** - Attention needed soon
- 🟡 **Medium** - Monitor closely
- 🟢 **Low** - Informational

### Creating an Alert Rule

1. Click **Create Alert Rule**
2. Configure rule:

**Basic Information:**
- Rule name
- Description
- Severity level

**Conditions:**
- Metric to monitor (accuracy, error rate, drift score)
- Operator (>, <, =, >=, <=)
- Threshold value
- Aggregation (avg, max, min)
- Time window (5m, 1h, 1d)

**Example:**
```
Alert when:
  Average accuracy
  < 0.85
  over 1 hour
```

**Advanced Conditions (AND/OR logic):**
```
Alert when:
  (Accuracy < 0.85 OR Error rate > 0.05)
  AND
  Prediction volume > 100
```

**Notification Channels:**
- ✉️ Email
- 💬 Slack
- 📱 SMS
- 🌐 Webhook

**Recipients:**
- Add team members
- Specify escalation contacts

3. Click **Create Rule**

### Managing Active Alerts

**When an alert triggers:**

1. **Review Alert**
   - What triggered it?
   - Current metric value
   - Historical context

2. **Acknowledge**
   - Click **Acknowledge** button
   - Add notes about investigation
   - Assign to team member

3. **Investigate**
   - Check model performance
   - Review recent predictions
   - Look for drift or anomalies

4. **Resolve**
   - Fix the issue
   - Click **Resolve** button
   - Document resolution notes

5. **Or Mark as False Positive**
   - If alert not valid
   - Adjust threshold to prevent recurrence

### Alert History

View past alerts:
1. Go to **Alert History** tab
2. Filter by:
   - Date range
   - Model
   - Severity
   - Status

**Insights:**
- Alert frequency trends
- Common alert types
- Mean time to resolution
- False positive rate

### Notification Preferences

Customize how you receive alerts:

1. Go to **Settings** → **Notifications**
2. Configure preferences:
   - Email notifications (on/off)
   - Slack integration
   - SMS for critical alerts only
   - Quiet hours (no alerts)
   - Digest emails (daily summary)

---

## Evidence & Reports

Exports turn what the platform has computed into an artifact you can hand to someone — a
stakeholder, an auditor, or another system — with every number traceable back to real
computed data, not a template. Find them at **Governance & Evidence → Evidence & Reports**,
or drive them through the `/api/v1/export/*` API.

**Full guide: [Audit & Explanation Reports](/user-guide/reports/)** — report categories,
output formats, the dashboard and API walkthroughs, scheduled reports, and delivery options
all live there now.

!!! note "Endpoint path"
    Report generation lives under `/api/v1/export/*`, not `/api/v1/reports`. The latter is
    not available and returns `404`.

---

## User Settings

### Profile Settings

**Personal Information:**
1. Go to **Profile** → **Profile** tab
2. Update:
   - Full name
   - Email address
   - Job title
   - Department
   - Avatar image

### Security Settings

**Password:**
1. Go to **Profile** → **Security** tab
2. Click **Change Password**
3. Enter current and new password
4. Click **Update**

**Two-Factor Authentication (2FA):**
1. Go to **Security** tab
2. Click **Enable 2FA**
3. Scan QR code with authenticator app
4. Enter verification code
5. Save backup codes

**Active Sessions:**
- View all logged-in devices
- Revoke access from untrusted devices

### API Keys

Scoped, revocable keys (`wbx_live_...`) for the SDK, CI/CD, and the MCP server.

**Creating an API Key:**

1. Go to **Profile** → **API Keys** tab
2. Click **Generate New Key**
3. Enter a key name, scopes, and an optional expiry
4. Copy and save the key securely
5. **Important:** the key is shown only once

**Managing Keys:**

- View all keys with their prefix, scopes, and expiry
- See last used date before deciding whether a key is safe to revoke
- Revoke unused keys — revocation takes effect immediately

Key creation and revocation are admin-only, since a key is an organization-wide credential.
See [API Keys](/account/api-keys/) for the full reference, the REST endpoints, and how
keys differ from login tokens.

### Notification Preferences

**Email Notifications:**
- Alert notifications
- Weekly summary
- Product updates
- Security alerts

**Slack Integration:**
- Connect Slack workspace
- Choose channels
- Configure alert routing

**In-App Notifications:**
- Desktop notifications
- Badge counts
- Sound alerts

---

## Best Practices

### Model Management
✅ Use descriptive model names
✅ Add detailed descriptions
✅ Tag models by environment (dev/staging/prod)
✅ Keep baseline metrics updated
✅ Archive old model versions

### Monitoring
✅ Set appropriate sampling rates (0.1 for high volume)
✅ Log predictions immediately
✅ Include metadata for debugging
✅ Review dashboards daily
✅ Investigate anomalies promptly

### Alerts
✅ Start with higher thresholds, tune down
✅ Use clear, actionable alert names
✅ Set up escalation paths
✅ Document alert response procedures
✅ Review and prune unused alert rules

### Explanations
✅ Generate explanations for edge cases
✅ Document explanation findings
✅ Use SHAP for global understanding
✅ Use LIME for specific predictions
✅ Share explanations with stakeholders

### Compliance
✅ Run regular fairness audits
✅ Document bias mitigation steps
✅ Generate monthly compliance reports
✅ Keep audit trail complete
✅ Review and update policies

---

## Getting Help

### Documentation
- **Getting Started:** [Getting Started guide](/get-started/getting-started/)
- **SDK Guide:** [SDK Documentation](/sdk/)
- **API Reference:** [REST API Reference](/sdk/api-reference/)
- **API Keys:** [Issuing and revoking API keys](/account/api-keys/)
- **Plans & Limits:** [Plans and API allowances](/get-started/plans/)
- **FAQ:** [Frequently Asked Questions](/help/faq/)
- **Troubleshooting:** [Troubleshooting guide](/help/troubleshooting/)

### Governance & compliance
- [Trust Score](/user-guide/trust-score/)
- [AI Risk Register](/user-guide/risk-register/)
- [Governance Review Boards](/user-guide/governance/)

### Integration quick references
- [MCP Server](/integrations/mcp/)
- [TensorFlow](/integrations/tensorflow/)
- [Hugging Face](/integrations/huggingface/)
- [LangChain](/integrations/langchain/)

### Support
- **Email:** [support@whiteboxxai.com](mailto:support@whiteboxxai.com)
- **Community:** [community.whiteboxxai.com](https://community.whiteboxxai.com)
