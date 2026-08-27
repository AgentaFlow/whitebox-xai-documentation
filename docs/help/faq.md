# WhiteBoxXAI - Frequently Asked Questions

Find answers to common questions about WhiteBoxXAI.

---

## Table of Contents

1. [General Questions](#general-questions)
2. [Account & Authentication](#account--authentication)
3. [Model Management](#model-management)
4. [Predictions & Logging](#predictions--logging)
5. [Metrics & Monitoring](#metrics--monitoring)
6. [Drift Detection](#drift-detection)
7. [Explainability (XAI)](#explainability-xai)
8. [Bias & Fairness](#bias--fairness)
9. [LLM Monitoring](#llm-monitoring)
10. [Alerts](#alerts)
11. [Reports](#reports)
12. [Billing & Plans](#billing--plans)
13. [Technical Questions](#technical-questions)
14. [Troubleshooting](#troubleshooting)

---

## General Questions

### What is WhiteBoxXAI?

WhiteBoxXAI is an AI Observability and Explainability platform that helps you monitor, explain, and audit machine learning models in production. It provides real-time monitoring, drift detection, bias auditing, and regulatory compliance tools.

### Who should use WhiteBoxXAI?

- **Data Scientists** - Monitor model performance and debug issues
- **ML Engineers** - Ensure production model reliability
- **Compliance Officers** - Audit models for regulatory compliance
- **Product Managers** - Understand AI impact on business metrics
- **Executives** - Get visibility into AI systems

### What types of models does WhiteBoxXAI support?

- **Classification** (Binary, Multi-class)
- **Regression** (Linear, Non-linear)
- **Clustering** (K-means, DBSCAN, etc.)
- **LLMs** (GPT, Claude, Llama, etc.)
- **Computer Vision** (Coming soon)
- **Recommendation Systems** (Coming soon)

### Which ML frameworks are supported?

- ✅ scikit-learn
- ✅ PyTorch
- ✅ TensorFlow/Keras
- ✅ XGBoost
- ✅ LightGBM
- ✅ CatBoost
- ✅ OpenAI, Anthropic, HuggingFace (LLMs)
- ✅ Custom models (via REST API)

### Is WhiteBoxXAI cloud-based or on-premise?

**Cloud (SaaS):** Primary offering at https://app.whiteboxxai.com
**Self-Hosted:** Enterprise plan includes on-premise deployment
**Hybrid:** Cloud control plane with on-premise data processing

### How much does WhiteBoxXAI cost?

See [Billing & Plans](#billing--plans) section below for detailed pricing.

---

## Account & Authentication

### How do I create an account?

1. Go to https://app.whiteboxxai.com
2. Click "Sign Up"
3. Enter email and password
4. Verify email
5. Complete onboarding

Free 14-day trial, no credit card required.

### I forgot my password. How do I reset it?

1. Go to login page
2. Click "Forgot Password?"
3. Enter your email
4. Check inbox for reset link
5. Create new password

### How do I enable two-factor authentication (2FA)?

1. Go to **Profile** → **Security**
2. Click **"Enable Two-Factor Authentication"**
3. Scan QR code with authenticator app (Google Authenticator, Authy, etc.)
4. Enter 6-digit code to verify
5. **Save backup codes** - Download or print them immediately
6. Click **"Verify and Enable"**

**Recommended authenticator apps:**
- Google Authenticator (iOS/Android)
- Authy (iOS/Android/Desktop)
- Microsoft Authenticator (iOS/Android)
- 1Password (cross-platform with auto-fill)
- Bitwarden Authenticator

**Why enable 2FA?**
- ✅ Protects against password breaches
- ✅ Required for compliance (SOC 2, HIPAA)
- ✅ Secures access to production models and data
- ✅ Recommended for all admin accounts

See [Two-Factor Authentication](/account/two-factor-authentication/) for the complete setup guide.

### Can I use SSO (Single Sign-On)?

Yes, on Enterprise plans:
- SAML 2.0
- OAuth 2.0
- LDAP/Active Directory
- Okta, Auth0, Azure AD

Contact sales@whiteboxxai.com to enable.

### How do I generate an API key?

1. Go to **Profile** → **API Keys**
2. Click **"Generate New Key"**
3. Enter a key name, scopes, and an optional expiry
4. If 2FA is enabled, enter your current 6-digit code
5. Copy the key — shown only once
6. Store it securely

Keys look like `wbx_live_...`. Creation and revocation are admin-only, since a key is an
organization-wide credential. Revoking a key takes effect immediately. Full reference:
[API Keys](/account/api-keys/).

**Security tips:**
- Use separate keys for dev/staging/prod, and one per consumer
- Set an expiry (`expires_in_days`) for temporary access
- Check `last_used_at` before revoking, and revoke anything unused
- Never commit keys to Git
- Use environment variables or a secrets manager

**Note:** API keys work independently of 2FA. Once generated, they don't require 2FA codes for SDK/API requests. 2FA only protects web login.

### I lost access to my authenticator app. How do I log in?

**If you have backup codes:**
1. Go to login page
2. Enter email and password
3. Click **"Use backup code"** link
4. Enter one of your backup codes
5. Each code works only once
6. Immediately set up new authenticator in Profile → Security

**If you don't have backup codes:**
1. Email support@whiteboxxai.com from your registered email
2. Include:
   - Account email
   - Company name
   - Reason for access loss
3. Identity verification required
4. Response time: 24-48 hours

**Prevention:**
- Store backup codes in password manager
- Use Authy (cloud backup feature)
- Keep backup codes in secure physical location

### How do I disable 2FA?

1. Go to **Profile** → **Security**
2. Enter your current 6-digit code
3. Click **"Disable Two-Factor Authentication"**
4. Enter your password to confirm
5. Your backup codes are invalidated

**Note:** We strongly recommend keeping 2FA enabled for security.

### How do I regenerate backup codes?

1. Go to **Profile** → **Security**
2. In the **Two-Factor Authentication** section
3. Click **"Regenerate Backup Codes"**
4. Enter your current 6-digit code
5. Old codes are invalidated
6. Download/print new codes immediately

**When to regenerate:**
- You've used most of your codes
- You suspect codes were compromised
- As part of regular security audit

### Can I use 2FA with multiple devices?

Yes! Two options:

**Option 1: Same secret on multiple devices**
- During setup, scan QR code with multiple authenticators
- All devices generate same codes
- Recommended for redundancy

**Option 2: Cloud-synced authenticator**
- Use Authy or 1Password
- Codes sync across devices automatically
- Easiest for multiple devices

**Not recommended:** Moving authenticator apps between devices without cloud sync (you'll lose access).

### How do I delete my account?

1. Go to **Profile** → **Account**
2. Scroll to **Danger Zone**
3. Click **"Delete Account"**
4. If 2FA is enabled, enter 6-digit code
5. Enter password to confirm
6. Data deleted within 30 days per GDPR

**Note:** This is permanent and cannot be undone.

---

## Model Management

### How do I register a model?

**Via Web UI:**
1. Go to **Models** → **Register Model**
2. Fill in model details
3. Click **Submit**

**Via SDK:**
```python
client.models.register(
    name="My Model",
    version="1.0.0",
    model_type="classification",
    features=["feature1", "feature2"]
)
```

See the [Getting Started guide](/get-started/getting-started/#registering-your-first-model) for details.

### What information do I need to register a model?

**Required:**
- Model name
- Model type (classification/regression/etc.)
- Version

**Recommended:**
- Features (input variables)
- Target variable
- Baseline metrics (accuracy, precision, etc.)
- Framework (scikit-learn, PyTorch, etc.)

**Optional:**
- Description
- Tags
- Training data size
- Training date

### Can I register multiple versions of the same model?

Yes! Best practice for version control:

```python
client.models.register(name="Fraud Detector", version="1.0.0")
client.models.register(name="Fraud Detector", version="1.1.0")
client.models.register(name="Fraud Detector", version="2.0.0")
```

Each version is tracked independently with its own metrics and predictions.

### How do I update model information?

1. Go to model details page
2. Click **"Edit Model"**
3. Update fields
4. Click **"Save Changes"**

**Via SDK:**
```python
client.models.update(
    model_id="model-uuid",
    description="Updated description",
    tags=["production", "v2"]
)
```

### Can I delete a model?

Yes, but **archiving** is recommended instead:

**Archive (recommended):**
- Preserves all historical data
- Stops monitoring
- Can be reactivated

**Delete:**
- Permanently removes model and ALL data
- Cannot be undone
- Use only if absolutely necessary

### How many models can I register?

**Free Trial:** Up to 3 models
**Starter:** Up to 10 models
**Professional:** Up to 50 models
**Enterprise:** Unlimited

---

## Predictions & Logging

### How do I log predictions?

**SDK (recommended):**
```python
client.predictions.log(
    model_id="model-uuid",
    input_data={"feature1": value1, "feature2": value2},
    output_data={"prediction": pred, "probability": prob}
)
```

**REST API:**
```bash
curl -X POST https://api.whiteboxxai.com/api/v1/predictions/log \
  -H "Authorization: Bearer TOKEN" \
  -d '{
    "model_id": "uuid",
    "inputs": {...},
    "output": {...}
  }'
```

See the [Getting Started guide](/get-started/getting-started/#logging-predictions) for examples.

### Do I need to log every prediction?

**High-volume models:** No, use sampling (0.01-0.1 = 1-10%)
**Low-volume models:** Yes, log all for complete visibility

Configure sampling rate when registering model or in model settings.

### What happens to my prediction data?

**Storage:**
- Encrypted at rest (AES-256)
- Stored in secure PostgreSQL database
- Backed up daily

**Retention:**
- **Free/Starter:** 30 days
- **Professional:** 90 days
- **Enterprise:** Custom (up to 2 years)

**Privacy:**
- Data isolated per account
- SOC 2 Type II compliant
- GDPR/CCPA compliant

### Can I log predictions in batches?

Yes! More efficient for high-volume scenarios:

```python
predictions = [
    {"inputs": {...}, "output": {...}},
    {"inputs": {...}, "output": {...}},
    # ... up to 1000 per batch
]

client.predictions.log_batch(
    model_id="model-uuid",
    predictions=predictions
)
```

**Limits:**
- Max 1,000 predictions per batch
- Max 10 MB payload size

### Should I include metadata?

Yes, recommended for:
- Customer/request IDs (for tracing)
- Timestamps
- Environment info (A/B test variant)
- Any context useful for debugging

**Example:**
```python
metadata={
    "customer_id": "CUST-12345",
    "experiment": "variant-B",
    "region": "US-West",
    "app_version": "2.3.1"
}
```

**Don't include:**
- PII (names, addresses, SSN)
- Sensitive data
- Unnecessary large objects

### What if my model doesn't return probabilities?

That's fine! Log what you have:

```python
# Classification without probabilities
output={"prediction": 1}

# Regression
output={"prediction": 42.5}

# Multi-class
output={"prediction": "category_A"}
```

Probabilities are optional but recommended for:
- Better confidence analysis
- Calibration monitoring
- Threshold optimization

---

## Metrics & Monitoring

### What metrics does WhiteBoxXAI track?

**Classification:**
- Accuracy
- Precision, Recall, F1
- AUC-ROC, AUC-PR
- Confusion Matrix
- Log Loss

**Regression:**
- MAE (Mean Absolute Error)
- MSE (Mean Squared Error)
- RMSE
- R² Score
- MAPE

**LLMs:**
- Token usage
- Cost per request
- Latency
- Toxicity scores
- RAG metrics

**System:**
- Prediction volume
- Response time
- Error rate
- Uptime

### How often are metrics updated?

- **Real-time dashboard:** Every 5 seconds (WebSocket)
- **Aggregated metrics:** Every 1 minute
- **Daily summaries:** Every 24 hours
- **Reports:** On-demand or scheduled

### Why are my metrics different from training?

**Common reasons:**

1. **Data Drift** - Production data differs from training data
2. **Sampling** - Not logging all predictions
3. **Class Imbalance** - Different distribution in production
4. **Label Lag** - Don't have ground truth yet
5. **Calculation Method** - Ensure same metric definitions

**To investigate:**
1. Check for drift in **Drift Detection**
2. Compare feature distributions
3. Verify ground truth labels are provided
4. Review sampling configuration

### How do I provide ground truth labels?

Labels are needed to calculate accuracy, precision, recall, etc.

**Method 1: Update prediction**
```python
client.predictions.update(
    prediction_id="pred-uuid",
    actual_label=1  # Ground truth
)
```

**Method 2: Batch update**
```python
client.predictions.update_batch([
    {"prediction_id": "uuid1", "actual_label": 0},
    {"prediction_id": "uuid2", "actual_label": 1},
])
```

**Method 3: Async via webhook**
Configure webhook in model settings to receive labels automatically.

### Can I track custom metrics?

Yes! On Professional and Enterprise plans:

```python
client.metrics.log_custom(
    model_id="model-uuid",
    metric_name="business_impact",
    metric_value=12500.0,
    timestamp=datetime.now()
)
```

Examples:
- Revenue impact
- Customer satisfaction
- Processing time
- Business KPIs

---

## Drift Detection

### What is drift?

**Data Drift:** Changes in input feature distributions
**Concept Drift:** Changes in relationship between features and target

**Example:**
- **Data Drift:** Average income of applicants increased from $50k to $75k
- **Concept Drift:** Same credit score now correlates differently with default risk

### How does drift detection work?

**Statistical tests:**
- **Continuous features:** Kolmogorov-Smirnov test
- **Categorical features:** Chi-squared test
- **Multivariate:** Maximum Mean Discrepancy (MMD)

**Comparison:**
- Production data vs. training data (baseline)
- Recent data vs. historical data (rolling window)

### How is drift severity determined?

**Drift Score:** 0.0 to 1.0

- **0.0 - 0.1:** Low (minor variation)
- **0.1 - 0.2:** Medium (investigate)
- **0.2+:** High (action required)

**Severity also considers:**
- Number of drifted features
- Impact on predictions
- Rate of change

### When should I retrain my model?

**Indicators:**
1. **High drift detected** (score > 0.2)
2. **Performance degradation** (accuracy drops)
3. **Multiple features drifting** (3+)
4. **Sustained drift** (not temporary)

**Best practice:**
- Set up drift alerts
- Review monthly
- Retrain quarterly (minimum)
- Retrain immediately if high drift + performance drop

### Can I configure drift detection sensitivity?

Yes, in model settings:

```python
client.models.update(
    model_id="model-uuid",
    drift_config={
        "threshold": 0.15,  # Lower = more sensitive
        "detection_frequency": "daily",  # hourly, daily, weekly
        "reference_period": "30d"  # 7d, 30d, 90d
    }
)
```

### How much data is needed for drift detection?

**Minimum:**
- 100 predictions
- 24 hours of data

**Recommended:**
- 1,000+ predictions
- 7 days of data

**Optimal:**
- 10,000+ predictions
- 30 days of data

More data = more reliable detection.

---

## Explainability (XAI)

### What is explainability?

**Explainability (XAI)** helps you understand:
- **Why** a model made a specific prediction
- **Which** features influenced the decision
- **How much** each feature contributed

### What explanation methods are available?

**SHAP (SHapley Additive exPlanations):**
- Game theory-based
- Globally consistent
- Works with any model
- Gold standard for explainability

**LIME (Local Interpretable Model-agnostic Explanations):**
- Local approximation
- Fast and interpretable
- Good for complex models

**Feature Importance:**
- Global view of feature impact
- Averaged across all predictions
- Useful for model understanding

### When should I use SHAP vs LIME?

**Use SHAP when:**
- Need accurate, theoretically sound explanations
- Comparing explanations across predictions
- Regulatory compliance (most trusted)
- Global feature importance

**Use LIME when:**
- Need fast explanations
- Working with very complex models
- Local understanding is sufficient
- Resource constraints

### How do I generate an explanation?

**From UI:**
1. Go to model's **Predictions** tab
2. Click on a prediction
3. Click **"Generate Explanation"**
4. Select method (SHAP/LIME)
5. Wait ~5-30 seconds

**Via SDK:**
```python
explanation = client.explanations.generate(
    model_id="model-uuid",
    instance={"feature1": 1.5, "feature2": "value"},
    method="shap"  # or "lime"
)

print(explanation["feature_weights"])
```

### How long does it take to generate explanations?

**Typical times:**
- **SHAP:** 10-30 seconds
- **LIME:** 5-15 seconds

**Factors:**
- Model complexity
- Number of features
- Prediction complexity

**Optimization:**
- Generate explanations asynchronously
- Request for important predictions only (high-value, contested, audited)

### Are explanations stored?

Yes, automatically:
- All generated explanations saved
- Searchable by model/prediction/date
- Exportable for compliance
- Subject to same retention policy as predictions

---

## Bias & Fairness

### What is bias in ML models?

**Bias** occurs when a model performs differently across demographic groups, leading to unfair outcomes.

**Example:**
A loan approval model that approves 70% of male applicants but only 50% of equally qualified female applicants is biased.

### What fairness metrics does WhiteBoxXAI measure?

**Demographic Parity:**
- Positive prediction rates should be similar across groups
- Example: Approval rates equal for all genders

**Equal Opportunity:**
- True positive rates should be equal
- Example: Qualified applicants approved at same rate

**Equalized Odds:**
- Both TPR and FPR should be equal
- Strictest fairness criterion

**Disparate Impact:**
- Ratio of positive outcomes between groups
- Legal standard (must be > 0.8 in US)

### How do I run a fairness audit?

1. Go to **Bias & Fairness**
2. Click **"Run Audit"**
3. Select:
   - Model
   - Protected attributes (gender, race, age, etc.)
   - Reference group (e.g., "male" for gender)
   - Fairness metrics
4. Click **"Run Audit"**
5. Review results in ~1-3 minutes

### What are "protected attributes"?

**Protected attributes** are characteristics that should not unfairly influence predictions:

**Common:**
- Race/Ethnicity
- Gender/Sex
- Age
- Religion
- Disability Status
- National Origin
- Marital Status

**Legal basis:**
- US: Title VII, ECOA, Fair Housing Act
- EU: GDPR Article 9
- Various state/country laws

### What if I detect bias?

**Steps:**

1. **Document** - Save audit report, share with team
2. **Investigate** - Root cause analysis
3. **Mitigate** - Apply fairness interventions
4. **Re-audit** - Verify improvements
5. **Monitor** - Set up ongoing fairness monitoring

**Mitigation strategies:**
- Rebalance training data
- Remove biased features
- Use fairness constraints
- Adjust decision thresholds

### Can I monitor fairness continuously?

Yes! Set up fairness alerts:

1. Go to **Alerts** → **Create Rule**
2. Select:
   ```
   Metric: Fairness Score
   Condition: < 80
   Frequency: Daily
   ```
3. Get notified if fairness drops

**Recommended:** Run full audit monthly, monitor score daily.

### Should I remove protected attributes from my model?

**Common misconception:** Removing protected attributes ensures fairness.

**Reality:** Not sufficient!

**Why:**
- Other features may correlate with protected attributes (proxies)
- Example: ZIP code correlates with race
- Model can still learn bias indirectly

**Better approach:**
1. Use fairness-aware training
2. Audit for bias regularly
3. Ensure training data is balanced
4. Monitor fairness metrics

---

## LLM Monitoring

### What LLM providers are supported?

- ✅ OpenAI (GPT-3.5, GPT-4, GPT-4 Turbo)
- ✅ Anthropic (Claude 2, Claude 3)
- ✅ Google (PaLM, Gemini)
- ✅ Cohere
- ✅ HuggingFace
- ✅ Azure OpenAI
- ✅ Self-hosted (Llama, Mistral, etc.)

### How do I monitor LLM calls?

**Wrap your LLM client:**

```python
from whiteboxxai.integrations.openai import wrap_openai

# Wrap OpenAI client
client = wrap_openai(
    openai_client,
    whiteboxxai_client=whiteboxxai_client,
    model_id="llm-model-uuid"
)

# Use normally - monitoring happens automatically
response = client.chat.completions.create(
    model="gpt-4",
    messages=[{"role": "user", "content": "Hello!"}]
)
```

**Or log manually:**

```python
whiteboxxai_client.llm.log(
    model_id="model-uuid",
    prompt="User question",
    completion="LLM response",
    metadata={
        "model": "gpt-4",
        "tokens": 150,
        "cost": 0.0045
    }
)
```

### What LLM metrics are tracked?

**Usage:**
- Total completions
- Input tokens
- Output tokens
- Total tokens

**Cost:**
- Per request cost
- Total spend
- Cost by model
- Cost trends

**Performance:**
- Latency (response time)
- Tokens per second
- Error rate

**Safety:**
- Toxicity scores
- PII detection
- Prompt injections
- Jailbreak attempts

**Quality (if using RAG):**
- Retrieval precision
- Answer relevance
- Faithfulness
- Context utilization

### How is LLM cost calculated?

Based on provider pricing:

**OpenAI GPT-4:**
- Input: $0.03 / 1K tokens
- Output: $0.06 / 1K tokens

**Example:**
```
Prompt: 500 tokens → $0.015
Completion: 300 tokens → $0.018
Total: $0.033
```

**Cost tracking:**
- Real-time spend monitoring
- Daily/weekly/monthly totals
- Budget alerts
- Cost optimization suggestions

### What is toxicity scoring?

**Toxicity detection** scans LLM outputs for harmful content:

**Categories:**
- Toxicity (general)
- Severe toxicity
- Obscenity
- Threats
- Insults
- Identity attacks

**Score:** 0.0 to 1.0 (higher = more toxic)

**Thresholds:**
- **0.0-0.3:** Safe
- **0.3-0.6:** Review
- **0.6-1.0:** High risk

**Action:**
- High-risk completions flagged
- Alerts triggered
- Review required

### Can I set budget limits for LLM costs?

Yes!

1. Go to **LLM Monitoring** → **Budget**
2. Set monthly budget (e.g., $1,000)
3. Configure alerts:
   - 50% spent
   - 75% spent
   - 90% spent
4. Optionally: Hard limit (stop calls when exceeded)

**Rate limiting:**
- Max requests per minute
- Max tokens per day
- Per-user limits

---

## Alerts

### What can I set alerts for?

**Performance:**
- Accuracy drops
- Error rate increases
- Latency spikes

**Drift:**
- Data drift detected
- Concept drift detected
- Specific features drifting

**Fairness:**
- Fairness score drops
- Demographic parity violated
- Equal opportunity violated

**Volume:**
- Prediction volume drops
- Unusual traffic spikes

**LLM:**
- Cost exceeds budget
- High toxicity responses
- Latency increases

**System:**
- API errors
- Integration failures
- Service downtime

### How do I create an alert?

1. Go to **Alerts** → **Create Rule**
2. Configure:
   - **Metric** - What to monitor
   - **Condition** - When to alert (>, <, =)
   - **Threshold** - Value that triggers alert
   - **Window** - Time period (5m, 1h, 1d)
   - **Severity** - Critical, High, Medium, Low
   - **Notifications** - Email, Slack, SMS, Webhook

3. Click **"Create Rule"**

**Example:**
```
Alert when:
  Accuracy < 0.85
  over 1 hour

Severity: High
Notify: team@company.com, #ml-alerts
```

### What notification channels are supported?

- ✉️ **Email** - Free on all plans
- 💬 **Slack** - Professional and Enterprise
- 📱 **SMS** - Enterprise only
- 📞 **PagerDuty** - Enterprise only
- 🌐 **Webhook** - All plans (custom integrations)
- 📱 **Microsoft Teams** - Enterprise only

### How do I integrate with Slack?

1. Go to **Settings** → **Integrations**
2. Click **"Connect Slack"**
3. Authorize WhiteBoxXAI app
4. Select workspace and channels
5. Configure:
   - Which alerts to send
   - Severity filter (send High+ only)
   - Quiet hours

**Alert format in Slack:**
```
🔴 Critical Alert: Low Accuracy

Model: Fraud Detector v2
Metric: Accuracy = 0.82 (threshold: 0.85)
Time: Last 1 hour

[View Dashboard] [Acknowledge] [Mute]
```

### Can I pause alerts temporarily?

Yes, **Maintenance Mode:**

1. Go to **Alerts** → **Settings**
2. Enable **Maintenance Mode**
3. Set duration (1h, 4h, 8h, 24h, custom)
4. All alerts suppressed during this time

**Use cases:**
- During deployments
- Scheduled maintenance
- Testing
- Known issues

### What is alert fatigue and how do I avoid it?

**Alert fatigue** = Too many alerts → ignore → miss important ones

**Prevention strategies:**

1. **Set appropriate thresholds**
   - Start conservative (high thresholds)
   - Tune based on false positives

2. **Use severity levels**
   - Reserve "Critical" for urgent issues
   - Use "Low" for informational

3. **Aggregate related alerts**
   - Don't alert on every occurrence
   - Alert if >5 in 10 minutes

4. **Quiet hours**
   - Suppress low-severity at night
   - Keep critical alerts always-on

5. **Regular review**
   - Disable unused rules
   - Adjust noisy alerts

---

## Reports

The full guide — report categories, output formats, the dashboard and API walkthroughs,
scheduling, delivery, retention, and branding — now lives at
[Audit & Explanation Reports](/user-guide/reports/). A couple of quick answers that come up
often:

### How do I generate a report?

Go to **Governance & Evidence → Evidence & Reports → Generate Report**, pick a template,
the models and date range, and a format. From the API it's
`POST /api/v1/export/exports` — see [Exports &
Reports](/sdk/api-reference/#exports--reports). Note the path is `/export/*`, not
`/reports`.

### Can I schedule reports?

Yes — generate a report once, click **Schedule**, and set a frequency, recipients, and
format. See [Scheduled reports](/user-guide/reports/#scheduled-reports) for the full
walkthrough and the API equivalent.

---

## Billing & Plans

### What plans are available?

**Demo (free):**
- Read-only showcase with preloaded sample data
- Unmetered — browse freely
- Every premium feature viewable

**Free ($0 forever):**
- 1,000 API calls per month
- No credit card required
- Full monitoring, drift detection, and SHAP/LIME explainability
- No premium features, no dedicated workspace

**Business Cloud ($599/month):**
- Dedicated workspace on its own subdomain, with isolated data
- Higher included API allowance, with pay-as-you-grow billing past it
- More CPU, GPU, and memory
- All premium features, including AI-driven architecture review
- GRC and audit logging; SSO and RBAC available
- Dedicated support and SLAs

**Enterprise Edition (custom, lifetime license):**
- Deployed on your own cloud or data center
- Air-gapped deployment available
- ISO 42001, NIST AI RMF, GDPR, EU AI Act, and CCPA governance
- SSO, RBAC, audit trails
- Professional installation, support, and training

Full detail: [Plans & Limits](/get-started/plans/). Current pricing:
[whiteboxxai.com/pricing](https://whiteboxxai.com/pricing).

### How is billing calculated?

**Free:** nothing to bill. Requests past the 1,000-call monthly allowance are paused until
the next cycle.

**Business Cloud:** a $599/month subscription with an included monthly API allowance.
Requests beyond the allowance are **not blocked** — they go through, and the overage is
reported as metered usage and billed on your next invoice. A production pipeline shouldn't
stop logging predictions because it had a busy month.

**Enterprise Edition:** a lifetime license with optional yearly maintenance and support.
Not metered.

Your current usage and remaining included allowance are shown in your account settings.

### What payment methods are accepted?

- 💳 Credit/debit cards (Visa, Mastercard, Amex)
- 🏦 ACH bank transfer (US only)
- 📄 Invoice (Enterprise only, NET-30)
- 💰 Wire transfer (Enterprise only)

Processed securely via Stripe.

### Can I change plans?

Yes, anytime:

**Upgrade:**
- Immediate access to new features
- Prorated charge for remaining month

**Downgrade:**
- Takes effect at next billing cycle
- Keep current features until then

**Cancel:**
- Access continues until end of billing period
- Data retained for 30 days
- No partial refunds (per ToS)

### Is there a discount for annual billing?

Contact **[sales@whiteboxxai.com](mailto:sales@whiteboxxai.com)** to discuss annual billing
or an Enterprise Edition license.

### What happens if I exceed my plan limits?

**On the Free plan:** past the 1,000-call monthly allowance, further API requests are paused
until the next cycle. Your existing data and dashboards stay fully accessible — nothing is
deleted, and reads keep working. Upgrade at any time to resume immediately.

**On Business Cloud:** requests past your included allowance are **allowed through**, not
blocked. The overage is reported as metered usage and appears on your next invoice.

**Enterprise Edition** isn't metered.

Check your current usage and remaining allowance in your account settings at any time. See
[Plans & Limits](/get-started/plans/#understanding-api-usage) for ways to reduce your call
volume — sampling, batching, and offline buffering all help.

---

## Technical Questions

### What APIs are available?

**REST API:**
- Model management
- Prediction logging
- Metrics retrieval
- Drift detection and bias/fairness auditing
- Explanations (SHAP/LIME)
- Trust Score and Risk Register
- Governance review boards
- Export and report generation
- Alert rule and instance management

**WebSocket:**
- `/api/v1/dashboard/ws` for live dashboard updates

**MCP server:**
- Use WhiteBoxXAI as a tool from Claude Desktop, Claude Code, LangChain agents, or any
  MCP-compatible client — see [MCP Server](/integrations/mcp/)

**GraphQL (Beta):**
- Flexible queries
- Efficient data fetching

**Full docs:** [API Reference](/sdk/api-reference/)

### What SDK languages are supported?

Currently:
- ✅ **Python** (fully supported)
- 🚧 **JavaScript/TypeScript** (beta)
- 🚧 **Java** (coming soon)
- 🚧 **Go** (coming soon)

Request other languages: support@whiteboxxai.com

### What is the API rate limit?

**Depends on plan:**

**Starter:**
- 1,000 requests/hour
- 100 predictions/minute

**Professional:**
- 10,000 requests/hour
- 1,000 predictions/minute

**Enterprise:**
- Custom limits
- Burst capacity

**Rate limit headers:**
```
X-RateLimit-Limit: 10000
X-RateLimit-Remaining: 9847
X-RateLimit-Reset: 1638360000
```

### How do I handle rate limits?

**SDK handles automatically:**
- Exponential backoff
- Retry logic
- Batch requests

**Manual implementation:**
```python
import time

while True:
    try:
        client.predictions.log(...)
        break
    except RateLimitError as e:
        time.sleep(e.retry_after)
```

### Is there a webhook for receiving events?

Yes! Configure in **Settings** → **Webhooks**

**Event types:**
- `alert.triggered` - Alert fires
- `drift.detected` - Drift found
- `model.updated` - Model changed
- `prediction.logged` - New prediction
- `report.generated` - Report ready

**Webhook payload:**
```json
{
  "event": "alert.triggered",
  "timestamp": "2025-12-05T14:30:00Z",
  "data": {
    "alert_id": "uuid",
    "model_id": "uuid",
    "severity": "high",
    "message": "Accuracy dropped to 0.82"
  }
}
```

### What regions is WhiteBoxXAI available in?

**Cloud (SaaS):**
- 🇺🇸 US-East (Virginia) - Primary
- 🇪🇺 EU-West (Ireland) - GDPR compliance
- 🇦🇵 Asia-Pacific (Singapore) - Coming Q1 2025

**Self-hosted:**
- Deploy anywhere (Enterprise plan)

**Data residency:**
- Data stays in selected region
- No cross-region transfer
- Compliant with local regulations

### Is WhiteBoxXAI compliant with regulations?

WhiteBoxXAI's product scope is deliberately five frameworks: **ISO/IEC 42001, GDPR, CCPA, the
EU AI Act, and the NIST AI Risk Management Framework.** See [AI
Regulations](/account/ai-regulations/) for how each maps to platform features, and [Audit &
Explanation Reports](/user-guide/reports/) for the **Compliance** report category that
generates evidence against them.

HIPAA isn't in that list — see the scope note at the top of [AI
Regulations](/account/ai-regulations/) for why frameworks outside these five (HIPAA, the
Colorado AI Act, and others) appear on that page as background context, not as something
WhiteBoxXAI targets or certifies against.

**Audit trail:**
- All actions logged
- Immutable records
- Exportable for compliance

---

## Troubleshooting

### My predictions aren't showing up

**Check:**

1. **Correct model ID?**
   ```python
   print(client.models.list())  # Find your model
   ```

2. **API key valid?**
   ```python
   print(client.auth.verify())  # Check auth
   ```

3. **Errors in logs?**
   ```python
   response = client.predictions.log(...)
   print(response.success, response.error)
   ```

4. **Sampling rate too low?**
   - Check model settings
   - Increase from 0.01 to 0.1 or 1.0 for testing

5. **Wait 5-10 seconds** - Processing delay

### I'm getting "Unauthorized" errors

**Solutions:**

1. **Regenerate API key:**
   - Profile → API Keys → Generate New
   - Update in code

2. **Check key location:**
   ```python
   import os
   print(os.getenv("WHITEBOXXAI_API_KEY"))
   ```

3. **Verify permissions:**
   - Key must have required permissions
   - Check in API Keys page

4. **Token expired:**
   - JWT tokens expire in 24h
   - SDK refreshes automatically
   - If using raw API, obtain new token

### Drift detection isn't working

**Requirements:**

1. **Minimum data:**
   - 100 predictions minimum
   - 24 hours minimum
   - Recommended: 1,000 predictions, 7 days

2. **Baseline configured:**
   - Provide training data distribution OR
   - Wait for baseline period (7 days)

3. **Detection enabled:**
   - Check model settings
   - Drift detection toggle ON

4. **Features match:**
   - Logged features must match registered features
   - Check feature names (case-sensitive)

### Explanations are slow or timing out

**Optimizations:**

1. **Reduce feature count:**
   - Use feature selection
   - Remove constant features
   - Typically <50 features for fast SHAP

2. **Use LIME instead:**
   - Faster than SHAP
   - Good for >100 features

3. **Increase timeout:**
   ```python
   explanation = client.explanations.generate(
       prediction_id="uuid",
       method="shap",
       timeout=120  # 2 minutes
   )
   ```

4. **Generate async:**
   ```python
   task = client.explanations.generate_async(...)
   # Check status later
   status = client.explanations.get_task_status(task.id)
   ```

### My metrics look wrong

**Common issues:**

1. **Ground truth labels missing:**
   - Need actual labels for accuracy
   - Update predictions with labels

2. **Sampling bias:**
   - Non-uniform sampling can skew metrics
   - Use random sampling

3. **Time zone issues:**
   - Check timestamps are correct timezone
   - Use UTC (recommended)

4. **Metric definition:**
   - Verify same calculation as training
   - Macro vs micro averaging
   - Weighted vs unweighted

### I'm getting "Invalid verification code" when using 2FA

**Common causes:**

1. **Time synchronization issue:**
   - TOTP codes depend on accurate time
   - Check device clock is synchronized
   - Tolerance: ±30 seconds
   - Fix: Enable automatic time sync on device

2. **Wrong time zone:**
   - Device must use correct time zone
   - Or use network time (NTP)

3. **Code already used:**
   - Each code valid for 30 seconds
   - Wait for new code if you just used one
   - Don't reuse codes

4. **Wrong account:**
   - Check you're using code from correct account entry
   - Verify account email in authenticator

5. **Old QR code:**
   - If you recently reset 2FA, old codes won't work
   - Rescan new QR code

**Quick fix:**
```bash
# Linux/Mac - sync time
sudo ntpdate -s time.nist.gov

# Check current time
date
```

**Still not working?** Use a backup code instead, then regenerate 2FA setup.

### How do I contact support?

**Documentation:**
- Search docs: https://docs.whiteboxxai.com
- FAQ: This document
- API Reference: [API Reference](/sdk/api-reference/)

**Community:**
- Forum: https://community.whiteboxxai.com
- Discord: https://discord.gg/whiteboxxai
- Stack Overflow: Tag `whiteboxxai`

**Support Channels:**

**All Plans:**
- 📧 Email: support@whiteboxxai.com
- 💬 In-app chat: Click icon in dashboard
- 📚 Knowledge base: https://help.whiteboxxai.com

**Professional:**
- ⏱️ Priority email (<8 hour response)
- 💬 Slack Connect

**Enterprise:**
- ☎️ Phone support
- 👤 Dedicated CSM
- 🔧 Slack/Teams integration
- ⚡ <2 hour response SLA

---

## Still Have Questions?

### Can't find your answer?

1. **Search docs** - https://docs.whiteboxxai.com
2. **Ask community** - https://community.whiteboxxai.com
3. **Contact support** - support@whiteboxxai.com

### Want a demo?

Book a personalized demo: https://whiteboxxai.com/demo

### Found a bug?

Report on GitHub: https://github.com/whiteboxxai/issues

---

*Last Updated: December 30, 2025*
*Version: 1.1*
