# Getting Started with WhiteBoxXAI

Welcome! This tutorial will guide you through your first steps with WhiteBoxXAI, from account setup to monitoring your first model.

!!! note "Just need a compliance report, not the full SDK walkthrough?"
    Jump straight to [Audit & Explanation Reports](/user-guide/reports/) — no Python
    required, generated entirely from the dashboard. Come back here when you're ready to
    connect your own models.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Creating Your Account](#creating-your-account)
3. [Installing the SDK](#installing-the-sdk)
4. [Registering Your First Model](#registering-your-first-model)
5. [Logging Predictions](#logging-predictions)
6. [Viewing Metrics](#viewing-metrics)
7. [Setting Up Alerts](#setting-up-alerts)
8. [Generating Your First Report](#generating-your-first-report)
9. [Next Steps](#next-steps)

---

## Prerequisites

Before you begin, ensure you have:

✅ **Python 3.9 or higher** (for SDK usage)
✅ **A trained ML model** (scikit-learn, PyTorch, TensorFlow, or any framework)
✅ **Production predictions** to log (or sample data)
✅ **Basic understanding** of your model's features and predictions

**Optional:**
- Git (for cloning examples)
- Jupyter Notebook (for interactive tutorials)

---

## Creating Your Account

### Step 1: Sign Up

1. Navigate to **https://app.whiteboxxai.com**
2. Click the **"Sign Up"** button
3. Fill in the registration form:
   ```
   Full Name: Your Name
   Email: your.email@company.com
   Password: ••••••••••••
   Company: Your Company Name (optional)
   ```
4. Check **"I agree to the Terms of Service"**
5. Click **"Create Account"**

### Step 2: Verify Email

1. Check your inbox for verification email
2. Click the verification link
3. You'll be redirected to the login page

### Step 3: First Login

1. Enter your email and password
2. Click **"Log In"**
3. You'll land on the dashboard with a welcome modal

> **Note:** For enhanced security, you can enable Two-Factor Authentication (2FA) later in your profile settings. See the [Security Best Practices](#enabling-two-factor-authentication-recommended) section below.

### Step 4: First-Run Experience

A welcome modal offers you two things:

**A guided tour.** A walkthrough of the sidebar, stopping at each area to explain what it's
for — Models, Performance, Drift Detection, Explainability, Fairness, Alerts, and Compliance.
You can take it, skip it, or exit partway through. It won't reappear once you've finished or
skipped it.

**Sample data.** Optionally seed your workspace with realistic demo models, predictions,
drift events, and bias audits, so the dashboards have something in them while you're finding
your way around. Seeding runs in the background and takes a moment — the dashboard shows a
`seeding` state and switches to `ready` when it's done.

!!! tip "Sample data is separate from your real data"
    Seeded demo records are clearly marked and can be removed at any time from **Settings →
    Demo Data**, which is also where you re-seed if you want a clean set. If you'd rather
    start from a genuinely empty workspace, just skip it — WhiteBoxXAI never fabricates
    numbers in your dashboards, so an empty account shows empty states rather than fake
    metrics.

If you'd rather drive this from the API:

```bash
# Where am I in onboarding?
GET  /api/v1/onboarding/status
# -> { "onboarding_completed": false, "demo_data_status": null, "model_count": 0 }

# Seed sample data
POST /api/v1/onboarding/seed-demo-data
# -> { "already_seeded": false, "demo_data_status": "seeding" }

# Mark the tour finished or skipped
POST /api/v1/onboarding/complete
```

`demo_data_status` is `seeding`, `ready`, `failed`, or `null` if never requested.

---

## Enabling Two-Factor Authentication (Recommended)

For enhanced account security, we strongly recommend enabling Two-Factor Authentication (2FA).

### Why Enable 2FA?

✅ **Enhanced Security** - Protects against password breaches
✅ **Compliance** - Required for many regulatory frameworks
✅ **Peace of Mind** - Secure access to your ML models and data

### Setting Up 2FA

1. Log in to WhiteBoxXAI
2. Click your profile icon → **Profile**
3. Scroll to the **Security** section
4. Click **"Enable Two-Factor Authentication"**

**Setup Process:**

1. **Scan QR Code:**
   - Open an authenticator app (Google Authenticator, Authy, 1Password, etc.)
   - Scan the displayed QR code
   - The app will generate a 6-digit code

2. **Verify Code:**
   - Enter the 6-digit code from your authenticator app
   - Click **"Verify and Enable"**

3. **Save Backup Codes:**
   - Download or print your backup codes
   - Store them securely (password manager or secure location)
   - Each code can be used once if you lose access to your authenticator

**2FA is now enabled!** Next login will require:
1. Your password
2. 6-digit code from your authenticator app

### Using 2FA on Login

When 2FA is enabled:

1. Enter email and password as usual
2. You'll see a **"Two-Factor Authentication"** prompt
3. Open your authenticator app
4. Enter the current 6-digit code
5. Click **"Verify"**

**Lost your authenticator?** Use a backup code instead of the 6-digit code.

> **Note:** API keys work independently of 2FA. Your existing integrations continue working without code changes.

---

## Installing the SDK

The WhiteBoxXAI SDK makes integration simple.

### Installation

Open your terminal and run:

```bash
pip install whitebox-xai-sdk
```

**For specific ML frameworks:**

```bash
# With scikit-learn support
pip install whitebox-xai-sdk[sklearn]

# With PyTorch support
pip install whitebox-xai-sdk[pytorch]

# With all integrations
pip install whitebox-xai-sdk[all]
```

### Verify Installation

```python
import whiteboxxai
print(whiteboxxai.__version__)
# Output: 1.0.0
```

### Get Your API Key

1. Log in to WhiteBoxXAI
2. Go to **Profile** → **API Keys**
3. Click **"Generate New Key"**
4. Enter a name: `"My Development Key"`
5. Select scopes: **read** and **write** (for getting started)
6. Leave the expiry blank for a key that doesn't expire, or set one
7. Click **"Generate"**
8. **Copy the key immediately** — it's shown only once

Your key looks like `wbx_live_...`. Key creation is admin-only, so if you don't see the
option, ask an organization admin to issue one for you. See [API
Keys](/account/api-keys/) for the full reference, including how to issue and revoke keys
through the API.

### Configure SDK

**Option 1: Environment Variable (Recommended)**

```bash
# Linux/Mac
export WHITEBOXXAI_API_KEY="your-api-key-here"

# Windows PowerShell
$env:WHITEBOXXAI_API_KEY="your-api-key-here"

# Windows Command Prompt
set WHITEBOXXAI_API_KEY=your-api-key-here
```

**Option 2: In Code**

```python
from whiteboxxai import WhiteBoxXAI

client = WhiteBoxXAI(api_key="your-api-key-here")
```

**Option 3: Config File**

Create `~/.whiteboxxai/config.json`:

```json
{
  "api_key": "your-api-key-here",
  "api_url": "https://api.whiteboxxai.com"
}
```

---

## Registering Your First Model

### Step 1: Prepare Model Information

Gather these details about your model:
- Model name and version
- Type (classification, regression, etc.)
- Features (input variables)
- Target variable (what it predicts)
- Performance metrics (accuracy, precision, etc.)

### Step 2: Register via Web UI

1. Log in to WhiteBoxXAI
2. Click **"Models"** in the sidebar
3. Click **"Register Model"** button

Fill in the form:

**Basic Information:**
```
Name: Credit Risk Model
Version: 1.0.0
Type: Binary Classification
Description: Predicts credit default risk
Framework: scikit-learn
```

**Features (comma-separated):**
```
credit_score, income, debt_ratio, age,
employment_length, loan_amount, payment_history
```

**Target:**
```
Target Variable: default_risk
Classes: 0 (no default), 1 (default)
```

**Baseline Metrics:**
```
Accuracy: 0.87
Precision: 0.82
Recall: 0.79
F1 Score: 0.80
AUC-ROC: 0.91
```

**Monitoring Settings:**
```
Sampling Rate: 0.1 (log 10% of predictions)
Drift Threshold: 0.1
Enable Monitoring: ✓ (checked)
```

Click **"Register Model"**

You'll see a success message with your **Model ID**: `550e8400-e29b-41d4-a716-446655440000`

**Save this ID!** You'll need it for logging predictions.

### Step 3: Register via SDK (Alternative)

```python
from whiteboxxai import WhiteBoxXAI

client = WhiteBoxXAI()

model = client.models.register(
    name="Credit Risk Model",
    version="1.0.0",
    model_type="classification",
    description="Predicts credit default risk",
    framework="scikit-learn",
    features=[
        "credit_score", "income", "debt_ratio", "age",
        "employment_length", "loan_amount", "payment_history"
    ],
    target="default_risk",
    baseline_metrics={
        "accuracy": 0.87,
        "precision": 0.82,
        "recall": 0.79,
        "f1_score": 0.80,
        "auc_roc": 0.91
    }
)

print(f"Model registered! ID: {model.id}")
```

---

## Logging Predictions

Now let's send predictions to WhiteBoxXAI for monitoring.

### Method 1: Manual Logging

Log individual predictions as they occur:

```python
from whiteboxxai import WhiteBoxXAI

client = WhiteBoxXAI()

# After your model makes a prediction
prediction = model.predict(features)

# Log to WhiteBoxXAI
client.predictions.log(
    model_id="550e8400-e29b-41d4-a716-446655440000",
    input_data={
        "credit_score": 720,
        "income": 75000,
        "debt_ratio": 0.35,
        "age": 42,
        "employment_length": 8,
        "loan_amount": 25000,
        "payment_history": 0.95
    },
    output_data={
        "prediction": 0,  # No default
        "probability": 0.12  # 12% risk
    },
    metadata={
        "customer_id": "CUST-12345",
        "application_date": "2025-12-05"
    }
)

print("Prediction logged successfully!")
```

### Method 2: Decorator Pattern

Automatically log predictions by decorating your prediction function:

```python
from whiteboxxai import WhiteBoxXAI
from whiteboxxai.decorators import log_predictions

client = WhiteBoxXAI()

@log_predictions(
    client=client,
    model_id="550e8400-e29b-41d4-a716-446655440000"
)
def predict_credit_risk(features):
    """Your prediction function"""
    prediction = model.predict([features])
    probability = model.predict_proba([features])[0][1]

    return {
        "prediction": int(prediction[0]),
        "probability": float(probability)
    }

# Use normally - logging happens automatically
result = predict_credit_risk({
    "credit_score": 720,
    "income": 75000,
    # ... other features
})
```

### Method 3: Batch Logging

Log multiple predictions at once:

```python
predictions = []

for customer in customers:
    features = extract_features(customer)
    pred = model.predict([features])

    predictions.append({
        "inputs": features,
        "output": {"prediction": int(pred[0])},
        "metadata": {"customer_id": customer.id}
    })

# Log all at once
client.predictions.log_batch(
    model_id="550e8400-e29b-41d4-a716-446655440000",
    predictions=predictions
)

print(f"Logged {len(predictions)} predictions")
```

### Method 4: Scikit-learn Integration

Wrap your scikit-learn model:

```python
from whiteboxxai.integrations.sklearn import wrap_model

# Wrap your trained model
monitored_model = wrap_model(
    model=my_sklearn_model,
    client=client,
    model_id="550e8400-e29b-41d4-a716-446655440000",
    feature_names=["credit_score", "income", ...]
)

# Use exactly like before - predictions logged automatically
prediction = monitored_model.predict(X_test)
probabilities = monitored_model.predict_proba(X_test)
```

---

## Viewing Metrics

### Dashboard Overview

1. Log in to WhiteBoxXAI
2. You'll see the main dashboard with:
   - **Total Predictions** - Count logged today
   - **Active Models** - Your registered models (should show 1)
   - **Prediction Volume Chart** - Predictions over time
   - **Recent Activity** - Latest predictions

### Model-Specific Metrics

1. Click **"Models"** in sidebar
2. Click on your model name: **"Credit Risk Model"**
3. View the **Overview** tab:
   - Current metrics (accuracy, precision, recall)
   - Predictions today/this week
   - Drift status
   - Last prediction timestamp

### Performance Trends

Click the **"Metrics"** tab to see:

**Time Series Charts:**
- Accuracy over time
- Precision over time
- Recall over time
- Prediction volume

**Filters:**
- Date range (Last 24h, 7 days, 30 days, Custom)
- Granularity (Hourly, Daily, Weekly)

**Export:**
- Download metrics as CSV
- Generate PDF report

### Prediction Log

Click the **"Predictions"** tab to see:

| Timestamp | Input Features | Prediction | Confidence | Status |
|-----------|----------------|------------|------------|--------|
| 2025-12-05 14:32 | credit_score: 720, income: 75k... | 0 (No default) | 0.88 | ✓ |
| 2025-12-05 14:30 | credit_score: 650, income: 55k... | 1 (Default) | 0.72 | ✓ |

**Actions:**
- Click any prediction to view details
- Generate explanation
- Filter by date/prediction value

---

## Setting Up Alerts

Stay informed when your model needs attention. Alert rules are created and managed in the
dashboard.

!!! note "Dashboard only for now"
    There's no public REST API for alerts yet — `/api/v1/alerts` returns `404`, and the SDK's
    `client.alerts.*` and `ModelMonitor.create_alert_rule()` call it, so they won't work.
    Create alert rules in the dashboard.

### Create Your First Alert

1. Go to **Observability → Alerts** in the sidebar
2. Click **"Create Alert Rule"**

**Configure Alert:**

```
Rule Name: Low Accuracy Alert
Description: Alert when accuracy drops below threshold

Condition:
  Metric: Accuracy
  Operator: <
  Threshold: 0.85
  Time Window: 1 hour

Severity: High

Notifications:
  ✓ Email: your.email@company.com
  ✓ Slack: #ml-alerts (if configured)

Model: Credit Risk Model
```

Click **"Create Rule"**

### Test Your Alert

The alert will trigger if average accuracy over the past hour drops below 0.85.

**To test:**
1. Wait for enough predictions (or generate test predictions)
2. Check the **Alerts** page for triggers
3. You'll receive an email notification when the alert fires

!!! tip "Drift and bias thresholds also feed your risk register"
    A drift report or bias audit that lands at `high` or `critical` severity automatically
    drafts an entry in your [AI Risk Register](/user-guide/risk-register/) — so a
    threshold breach becomes a tracked, owned risk rather than just a notification you
    might miss.

### Recommended Starter Alerts

**1. Performance Alert:**
```
Alert when: Accuracy < 0.85 over 1 hour
Severity: High
```

**2. Volume Alert:**
```
Alert when: Predictions < 10 over 1 hour
Severity: Medium
(Indicates integration issue or traffic drop)
```

**3. Drift Alert:**
```
Alert when: Drift Score > 0.2
Severity: High
```

**4. Error Rate Alert:**
```
Alert when: Error Rate > 0.05 over 30 minutes
Severity: Critical
```

---

## Generating Your First Report

1. Go to **Governance & Evidence → Evidence & Reports** in the sidebar.
2. Click **Generate Report**, pick a template (e.g. Performance Report), the models and
   date range to cover, and an output format.
3. Click **Generate Report** again to confirm — processing takes ~30–300 seconds — then
   **View Report**, **Download**, or **Share** it directly with a stakeholder.

That's the same thing you'd do from the API via `POST /api/v1/export/exports` (note the
path — `/api/v1/reports` returns `404`).

**Full guide: [Audit & Explanation Reports](/user-guide/reports/)** — report contents,
the API walkthrough, scheduled reports, and delivery options.

---

## Next Steps

Congratulations! You've successfully:

✅ Created an WhiteBoxXAI account
✅ Installed the SDK
✅ Registered your first model
✅ Logged predictions
✅ Viewed metrics in dashboard
✅ Set up alerts
✅ Generated your first report

### Continue Learning

**Explore Advanced Features:**
- 📚 [User Guide](/user-guide/) - Complete feature documentation
- 🔍 [Understanding Explanations](/user-guide/#understanding-explanations) - Understand model decisions
- 📊 [Detecting Drift](/user-guide/#detecting-drift) - Monitor data distribution changes
- ⚖️ [Auditing for Bias](/user-guide/#auditing-for-bias) - Check for bias
- 🤖 [Monitoring LLMs](/user-guide/#monitoring-llms) - If using language models

**Governance & Compliance:**
- 🎯 [Trust Score](/user-guide/trust-score/) - One 0–100 index per model
- 🛡️ [AI Risk Register](/user-guide/risk-register/) - Structured risk inventory
- ⚖️ [Governance Review Boards](/user-guide/governance/) - Multi-party approval workflows

**SDK & API:**
- [SDK Documentation](/sdk/) - Full SDK reference
- [API Reference](/sdk/api-reference/) - REST API endpoints
- [API Keys](/account/api-keys/) - Issuing and revoking credentials
- [MCP Server](/integrations/mcp/) - Use WhiteBoxXAI from non-Python clients and agents

**More help:**
- [Plans & Limits](/get-started/plans/) - API allowances and what each plan includes
- [Troubleshooting](/help/troubleshooting/) - Common issues
- [Two-Factor Authentication](/account/two-factor-authentication/) - 2FA setup and management

**Community:**
- Join discussions at https://community.whiteboxxai.com
- Watch tutorials at https://learn.whiteboxxai.com
- Follow us on Twitter @WhiteBoxXAI_io

---

## Common Questions

### How many predictions should I log?

**High-volume models (>10,000/day):**
- Use sampling rate 0.01-0.1 (1-10%)
- Still provides statistical significance
- Reduces costs

**Low-volume models (<1,000/day):**
- Use sampling rate 1.0 (100%)
- Log all predictions for complete visibility

### When will I see metrics?

- Metrics appear immediately after first predictions
- Trends need ~24-48 hours of data
- Drift detection needs ~7 days to establish baseline

### How do I integrate with existing code?

**Minimal changes required:**

```python
# Before
prediction = model.predict(features)

# After (one line added)
prediction = model.predict(features)
client.predictions.log(model_id, input_data=features, output_data=prediction)
```

**Or use decorator for zero code changes:**

```python
@log_predictions(client, model_id)
def predict(features):
    return model.predict(features)
```

### What about sensitive data?

**Best practices:**
- Don't log PII (names, addresses, SSN)
- Use customer IDs in metadata instead
- Hash sensitive features if needed
- Configure data retention policies

**Example:**

```python
client.predictions.log(
    model_id=model_id,
    input_data={
        "credit_score": 720,
        # Don't log: "customer_name": "John Doe"
        # Don't log: "ssn": "123-45-6789"
    },
    metadata={
        "customer_id": "CUST-12345"  # OK - non-PII identifier
    }
)
```

### How do I add team members?

1. Go to **Settings** → **Team**
2. Click **"Invite Member"**
3. Enter email and select role:
   - **Admin** - Full access
   - **Member** - View and edit
   - **Viewer** - Read-only
4. Click **"Send Invitation"**

**Security Tip:** For admin accounts, require 2FA to be enabled. You can check who has 2FA enabled in the Team management page.

---

## Troubleshooting

### "API key invalid" error

**Solution:**
1. Check your API key in Profile → API Keys — confirm it hasn't been revoked or expired
2. Ensure no extra spaces or newlines, and that you copied the whole `wbx_live_...` value
3. Issue a new key if needed (the old one can't be retrieved — only revoked)

```python
# Verify key is set
import os
print(os.getenv("WHITEBOXXAI_API_KEY"))
```

If you get a `403` instead of a `401`, the key is valid but its scopes don't cover the
operation — issue a new key with the scopes you need. See [API
Keys](/account/api-keys/#troubleshooting).

### Predictions not appearing

**Check:**
1. Model ID is correct
2. Predictions successfully sent (no errors)
3. Refresh dashboard (may take 5-10 seconds)

```python
# Verify prediction sent
response = client.predictions.log(...)
print(f"Success: {response.success}")
print(f"Prediction ID: {response.prediction_id}")
```

### SDK import error

```bash
# Reinstall SDK
pip uninstall whiteboxxai-sdk
pip install whitebox-xai-sdk

# Verify installation
pip show whiteboxxai-sdk
```

### Can't access account / Lost authenticator

**If you have backup codes:**
1. Go to login page
2. Enter email and password
3. Click **"Use backup code"** instead of entering 6-digit code
4. Enter one of your backup codes
5. Immediately go to Profile → Security and set up a new authenticator

**If you don't have backup codes:**
- Contact support at support@whiteboxxai.com
- Account recovery requires identity verification
- Response time: 24-48 hours

---

## Need Help?

**Documentation:**
- 📖 [User Guide](/user-guide/)
- 🔧 [API Reference](/sdk/api-reference/)
- ❓ [FAQ](/help/faq/)

**Support:**
- ✉️ Email: support@whiteboxxai.com
- 💬 Chat: Click chat icon in dashboard
- 🐛 SDK issues: [AgentaFlow/whitebox-python-sdk](https://github.com/AgentaFlow/whitebox-python-sdk/issues)

**Learning Resources:**
- 🎥 Video Tutorials: https://learn.whiteboxxai.com
- 📚 Blog: https://blog.whiteboxxai.com
- 👥 Community: https://community.whiteboxxai.com

---

*Happy Monitoring! 🚀*

*Last Updated: December 30, 2025*
