# Interactive Demos

This document catalogs interactive demonstrations that showcase WhiteBoxXAI platform capabilities through live, hands-on experiences.

## Overview

Interactive demos allow prospective users, customers, and stakeholders to experience WhiteBoxXAI's features firsthand without requiring installation or setup. These demos run in sandboxed environments with sample data and pre-configured models.

---

## 🎯 Demo Catalog

### Demo 1: Quick Start - First Prediction in 60 Seconds
**URL:** demo.whiteboxxai.example.com/quickstart
**Duration:** 60 seconds
**Audience:** All users
**Goal:** Show how easy it is to get started

#### Demo Flow

**Step 1: Welcome Screen (5 sec)**
```
Welcome to WhiteBoxXAI!

Let's log your first prediction and see an explanation.
No signup required - this is a live demo environment.

[Start Demo Button]
```

**Step 2: Pre-loaded Code (15 sec)**
```python
# Your model is already set up!
from whiteboxxai import WhiteBoxXAI

client = WhiteBoxXAI(api_key="demo")  # Demo API key
model_id = "demo-credit-model"      # Pre-registered model

# Sample customer data
customer = {
    "credit_score": 720,
    "income": 85000,
    "debt_ratio": 0.35,
    "employment_years": 5,
    "previous_defaults": 0
}

# Log prediction with explanation
result = client.predictions.log(
    model_id=model_id,
    features=customer,
    prediction={"default_risk": 0.18},  # 18% risk
    explanation_config={"method": "shap"}
)

print(f"Prediction logged! View at: {result['dashboard_url']}")
```

**[Click to Run Code]**

**Step 3: Code Execution (5 sec)**
```
Running...
✓ Prediction logged!
✓ Explanation generated!
```

**Step 4: Explanation View (35 sec)**
[Automatically redirect to dashboard showing:]

- **Prediction Details Card**
  - Customer ID: DEMO-12345
  - Prediction: 18% default risk ✅ LOW RISK
  - Confidence: 92%
  - Timestamp: [Current time]

- **SHAP Waterfall Plot** (Interactive)
  - Base value: 30% (average default rate)
  - credit_score=720: -8% ⬇️ (reduces risk)
  - income=$85k: -3% ⬇️ (reduces risk)
  - debt_ratio=0.35: -2% ⬇️ (reduces risk)
  - employment_years=5: +1% ⬆️ (slightly increases)
  - Final prediction: 18% ✅

- **Feature Values Table**
  | Feature | Value | Impact |
  |---------|-------|--------|
  | credit_score | 720 | -8% ⬇️ |
  | income | $85,000 | -3% ⬇️ |
  | debt_ratio | 0.35 | -2% ⬇️ |

**[Hover over bars for details]**
**[Click "Try Another Prediction" to modify values]**

**Step 5: Call to Action (5 sec)**
```
That's it! In 60 seconds you:
✓ Logged a prediction
✓ Generated an explanation
✓ Viewed results in dashboard

Ready to try with your own models?
[Sign Up Free] [View Documentation]
```

#### Technical Implementation

**Frontend:**
- React app with Monaco Editor (code editor)
- Recharts for SHAP visualization
- Live code execution in sandboxed iframe
- WebSocket for real-time updates

**Backend:**
- Dedicated demo API endpoint
- Rate limiting: 10 requests/minute per IP
- Isolated demo database
- Auto-cleanup after 1 hour

**Infrastructure:**
- Cloud-hosted demo environments
- Auto-scaling based on traffic
- CDN for fast global access

---

### Demo 2: Interactive Explainability Explorer
**URL:** demo.whiteboxxai.example.com/explainability
**Duration:** 3-5 minutes
**Audience:** Data scientists, analysts
**Goal:** Deep dive into explanation methods

#### Demo Flow

**Welcome Screen**
```
Explore Different Explanation Methods

Compare SHAP, LIME, and feature importance for the same prediction.
Use sliders to modify feature values and see how explanations change.

[Start Exploring]
```

**Interactive Interface**

**Left Panel: Feature Inputs (Sliders)**
```
Customer Profile - Adjust Values to See Impact

Credit Score: [300 ========●===== 850]  720
Income:       [$0 =======●====== $200k] $85k
Debt Ratio:   [0% =====●========= 100%]  35%
Age:          [18 ======●======== 80]    42
Employment:   [0 ====●========== 30yr]   5yr

[Generate Explanations Button]
```

**Center Panel: Explanation Views (Tabs)**

**Tab 1: SHAP Waterfall**
- Interactive waterfall plot
- Hover for detailed tooltips
- Click features to highlight

**Tab 2: SHAP Force Plot**
- Interactive force plot
- Drag features to reorder
- Zoom in/out

**Tab 3: LIME**
- Feature weights visualization
- Local approximation boundary
- Feature importance ranking

**Tab 4: Feature Importance**
- Global importance chart
- Permutation importance
- Comparison view

**Right Panel: Prediction Results**
```
Current Prediction

Default Risk: 18%
Risk Level: LOW ✅
Confidence: 92%

Recommendation:
✅ Approve loan

Key Drivers:
1. Excellent credit score (720)
2. Stable income ($85k)
3. Low debt ratio (35%)

Concerns:
⚠️ Relatively short employment (5yr)
```

**Bottom Panel: Scenarios**
```
Try These Scenarios:

[Excellent Candidate]  [Risky Candidate]  [Borderline Case]
[Reset to Default]     [Randomize]        [Clear]
```

**Pre-defined Scenarios:**

1. **Excellent Candidate**
   - Credit score: 800
   - Income: $150k
   - Debt ratio: 20%
   - Result: 5% risk ✅

2. **Risky Candidate**
   - Credit score: 550
   - Income: $35k
   - Debt ratio: 75%
   - Result: 68% risk ❌

3. **Borderline Case**
   - Credit score: 650
   - Income: $60k
   - Debt ratio: 50%
   - Result: 42% risk ⚠️

#### Features

- **Real-time Updates:** Explanations regenerate as sliders move
- **Comparison Mode:** View multiple scenarios side-by-side
- **Export:** Download explanations as PDF/PNG
- **Share:** Generate shareable link to current scenario

---

### Demo 3: Bias Detection Playground
**URL:** demo.whiteboxxai.example.com/bias
**Duration:** 5-7 minutes
**Audience:** Compliance officers, ethicists, data scientists
**Goal:** Demonstrate bias detection capabilities

#### Demo Flow

**Welcome Screen**
```
Bias Detection Playground

Analyze a pre-trained credit scoring model for fairness across demographic groups.
Discover how WhiteBoxXAI helps you build fair and compliant AI systems.

Dataset: 10,000 loan applications
Protected Attributes: Age, Gender, Race

[Analyze Fairness]
```

**Step 1: Select Protected Attributes**
```
Choose attributes to analyze:

☑️ Age (18-80)
☑️ Gender (Male, Female, Non-binary)
☑️ Race (White, Black, Hispanic, Asian, Other)
☐ Marital Status
☐ Geographic Location

[Run Analysis]
```

**Step 2: Fairness Metrics Dashboard**

**Overall Fairness Score: 72/100** ⚠️

**Metric Cards:**

**Demographic Parity**
```
Score: 68/100 ⚠️

Approval rates by group:
Male:     58%  ████████████████████
Female:   45%  ███████████████
Non-binary: 52% ██████████████████

Disparity: 13 percentage points
Status: Requires attention
```

**Equal Opportunity**
```
Score: 75/100 ⚠️

True positive rate (qualified applicants approved):
White:    82%  ██████████████████████
Black:    76%  ████████████████████
Hispanic: 79%  █████████████████████
Asian:    85%  ███████████████████████

Disparity: 9 percentage points
Status: Minor concerns
```

**Predictive Parity**
```
Score: 83/100 ✅

Precision by group:
Age 18-30: 88%  ████████████████████████
Age 31-50: 91%  ██████████████████████████
Age 51+:   89%  █████████████████████████

Disparity: 3 percentage points
Status: Acceptable
```

**Step 3: Interactive Visualizations**

**Chart 1: Approval Rate by Group (Bar Chart)**
- X-axis: Demographic groups
- Y-axis: Approval rate (%)
- Color: Green if within threshold, red if disparate
- Hover: Show exact numbers and confidence intervals

**Chart 2: False Positive/Negative Rates (Grouped Bar)**
- Compare error rates across groups
- Highlight concerning disparities
- Interactive legend

**Chart 3: Feature Impact by Group (Heatmap)**
- Rows: Features
- Columns: Demographic groups
- Color intensity: Average SHAP value
- Reveals which features impact different groups

**Step 4: Detailed Analysis**

**Click on any group for deep dive:**
```
Female Applicants - Detailed Analysis

Sample Size: 4,832
Approval Rate: 45% (vs 52% overall)
Avg Credit Score: 698 (vs 685 overall)
Avg Income: $67k (vs $65k overall)

Key Findings:
⚠️ Despite higher credit scores, approval rate is lower
⚠️ Model relies more heavily on "employment_years" for females
⚠️ 13 percentage point gap vs males

Potential Issues:
- Historical bias in training data
- Indirect discrimination through proxy features
- Different decision thresholds by group

Recommended Actions:
1. Investigate employment_years feature
2. Consider re-weighting training data
3. Apply fairness constraints during retraining
4. Monitor ongoing for fairness drift

[Generate Fairness Report]
```

**Step 5: Mitigation Strategies**

```
Simulate Bias Mitigation

Try different strategies to improve fairness:

○ Reweighting: Balance training data by protected groups
○ Threshold Optimization: Adjust decision thresholds per group
○ Fairness Constraints: Add fairness penalty to model training
○ Feature Removal: Drop potentially discriminatory features

[Apply Strategy]
```

**After applying strategy:**
```
Results After Reweighting:

Fairness Score: 72 → 87 (+15) ✅

Demographic Parity: 68 → 91 ✅
Equal Opportunity: 75 → 89 ✅
Predictive Parity: 83 → 82 ✅

Approval rates:
Male:     58% → 54%
Female:   45% → 52%
Non-binary: 52% → 53%

Trade-offs:
Accuracy: 85% → 83% (-2%)
F1 Score: 0.81 → 0.79 (-0.02)

Decision: Accept this mitigation?
[Yes, Apply] [No, Try Another]
```

---

### Demo 4: Real-Time Monitoring Dashboard
**URL:** demo.whiteboxxai.example.com/monitoring
**Duration:** Continuous (leave running)
**Audience:** ML engineers, DevOps
**Goal:** Show real-time monitoring capabilities

#### Demo Flow

**Live Dashboard** (Updates every 2 seconds with simulated data)

**Header:**
```
Live Model Monitoring
Model: fraud-detection-v2  |  Status: ✅ Healthy  |  Uptime: 99.98%
Last updated: 2 seconds ago  |  Predictions today: 142,847
```

**Metrics Grid:**

**Prediction Volume (Live Chart)**
- Line chart showing predictions/minute
- Last 60 minutes
- Animated as new data arrives
- Hover for exact numbers

**Accuracy Trend (Live Chart)**
- Rolling accuracy over time
- Alert threshold line at 90%
- Color changes if below threshold

**Latency Distribution (Live Histogram)**
- P50, P95, P99 markers
- Updates in real-time
- Target latency line

**Alert Feed (Scrolling)**
```
Recent Alerts:

⚠️ 14:23:15 - Data drift detected in "transaction_amount"
✅ 14:22:08 - Accuracy recovered to 92%
⚠️ 14:18:42 - High latency (P95: 450ms)
✅ 14:15:00 - Drift resolved in "merchant_category"
```

**Feature Drift Heatmap**
- Rows: Features
- Columns: Time windows (last 1h, 6h, 24h)
- Color: Green (no drift) → Red (high drift)
- Click for detailed analysis

**Prediction Distribution (Live)**
- Pie chart: Fraud vs Legitimate
- Updates as predictions come in
- Shows percentage and counts

**Model Health Score**
```
Overall Health: 94/100 ✅

Components:
Accuracy:      96/100 ✅
Latency:       88/100 ✅
Data Quality:  95/100 ✅
Drift:         92/100 ✅
Uptime:        99/100 ✅
```

**Interactive Elements:**
- **Pause/Resume:** Stop live updates
- **Speed Control:** 1x, 2x, 5x, 10x speed
- **Time Range:** Last 1h, 6h, 24h, 7d
- **Filters:** By prediction type, alert severity

**Simulate Incidents:**
```
Trigger Test Scenarios:

[Accuracy Drop]     - Simulate sudden accuracy decrease
[Data Drift]        - Inject drifted features
[High Latency]      - Simulate slow predictions
[Spike in Volume]   - Increase prediction rate 10x
[Recovery]          - Return to normal
```

---

### Demo 5: End-to-End ML Pipeline
**URL:** demo.whiteboxxai.example.com/pipeline
**Duration:** 10-15 minutes
**Audience:** ML engineers, data scientists
**Goal:** Show full integration workflow

#### Demo Flow

**Multi-step interactive demo showing:**

**Step 1: Train Model** (Jupyter Notebook style)
- Load data
- Train scikit-learn model
- Evaluate performance

**Step 2: Register with WhiteBoxXAI**
- Initialize SDK
- Register model
- Configure monitoring

**Step 3: Deploy Model**
- Create Flask API endpoint
- Add WhiteBoxXAI logging
- Test endpoint

**Step 4: Generate Traffic**
- Simulate production requests
- Watch predictions logged
- See explanations generated

**Step 5: Monitor Performance**
- View dashboard
- See metrics update
- Receive alerts

**Step 6: Detect Issues**
- Inject data drift
- See alerts fire
- Investigate root cause

**Step 7: Iterate**
- Retrain model
- Deploy new version
- Compare versions

---

## 🛠️ Demo Infrastructure

### Technical Architecture

**Frontend (React + TypeScript):**
- Next.js for server-side rendering
- Recharts for visualizations
- Monaco Editor for code editing
- TailwindCSS for styling

**Backend (Python FastAPI):**
- Demo-specific endpoints
- Sandboxed execution environment
- Pre-seeded database
- Rate limiting

**Database:**
- PostgreSQL with demo data
- Read-only for users
- Refreshes every hour

**Deployment:**
- Cloud-hosted environment
- Auto-scaling (2-20 replicas)
- CDN for static assets
- Edge caching for performance

### Security Considerations

**Isolation:**
- Each demo session in isolated container
- 15-minute timeout
- No access to production data
- No persistent storage

**Rate Limiting:**
- 10 requests/minute per IP
- 100 requests/hour per IP
- CAPTCHA for suspicious activity
- IP blocking for abuse

**Data Privacy:**
- All demo data is synthetic
- No PII collection
- No account required
- Analytics anonymized

### Performance Optimization

**Caching:**
- Pre-compute explanations
- Cache common scenarios
- CDN for static assets

**Preloading:**
- Warm up models on pod start
- Pre-generate sample data
- Prefetch common queries

**Scaling:**
- Horizontal pod autoscaling
- Load balancing across regions
- Graceful degradation

---

## 📊 Demo Analytics

### Track These Metrics

**Usage Metrics:**
- Demo sessions started
- Completion rate per demo
- Average time spent
- Most popular demos
- Geographic distribution

**Engagement Metrics:**
- Button clicks
- Feature interactions
- Scenario selections
- Code executions
- Dashboard views

**Conversion Metrics:**
- Sign-ups from demos
- Documentation clicks
- Sales inquiries
- Video views
- Workshop registrations

### A/B Testing

**Test Variations:**
- Demo length (short vs detailed)
- Starting scenario (simple vs complex)
- Explanation depth
- Visual design
- Call-to-action placement

---

## 🎨 Design Guidelines

### Visual Design

**Color Scheme:**
- Use WhiteBoxXAI brand colors
- Green for positive/good (high accuracy, low risk)
- Red for negative/concerning (drift, errors)
- Yellow for warnings
- Blue for neutral information

**Typography:**
- Headers: 24-32px, bold
- Body: 14-16px, regular
- Code: Monaco, 12-14px
- Ensure readability on all devices

**Layout:**
- Clean, uncluttered interface
- Generous whitespace
- Clear visual hierarchy
- Responsive design (mobile-friendly)

### User Experience

**Onboarding:**
- Clear instructions at each step
- Tooltips for unfamiliar terms
- Progress indicators
- Skip/restart options

**Interactivity:**
- Immediate feedback on actions
- Smooth animations (300ms)
- Loading states for async operations
- Error handling with helpful messages

**Accessibility:**
- WCAG 2.1 AA compliance
- Keyboard navigation
- Screen reader support
- High contrast mode option

---

## 🚀 Deployment Checklist

Before deploying a new demo:

- [ ] Test all interactions
- [ ] Verify data loads correctly
- [ ] Check performance (< 2s load time)
- [ ] Test on mobile devices
- [ ] Verify rate limiting works
- [ ] Check accessibility
- [ ] Test error scenarios
- [ ] Review analytics setup
- [ ] Load test (100 concurrent users)
- [ ] Security audit
- [ ] Monitor logs after deployment
- [ ] Create runbook for issues

---

## 📞 Support

For demo questions:
- **Email:** demos@whiteboxxai.example.com
- **Slack:** #demos
- **On-call:** For production demo issues

---

*Last Updated: December 2024*
*Version: 1.0*
