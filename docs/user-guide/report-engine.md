# Phase 6 Report Engine

The Phase 6 Report Engine is the cornerstone of WhiteBoxXAI's governance and auditing capabilities, delivering automated, compliance-ready reports for AI Risk Officers and Auditors.

## Overview

Designed explicitly for the enterprise AI Risk Officer, the Phase 6 Report Engine prioritizes **auditability and transparency**. It produces defensible evidence across five major compliance frameworks: ISO/IEC 42001, GDPR, CCPA, the EU AI Act, and NIST AI RMF.

### Key Principles
- **No Fabricated Metrics**: If a specific metric or explanation cannot be computed honestly, the engine explicitly states so or omits it. We never guess.
- **Plain-Language Verdicts**: Every raw ML output is accompanied by a plain-language summary.
- **Strict Tenant Isolation**: All reports are generated with rigid cross-tenant data separation.

## Supported Report Types

You can generate 9 distinct report types to satisfy various internal governance and external regulatory requirements:

1. **Model Card** (`model_card`): High-level overview of model purpose, intended use, and performance metrics.
2. **Fairness Audit** (`fairness_audit`): Detailed demographic parity and equal opportunity metrics across protected classes.
3. **Drift & Stability** (`drift_stability`): Concept drift and data drift statistics over time.
4. **Data Protection** (`data_protection`): Evidence of PII masking and data minimization.
5. **Security Posture** (`security_posture`): Penetration testing and adversarial robustness summaries.
6. **Compliance Evidence** (`compliance_evidence`): Cross-referenced evidence mapping directly to ISO 42001 and NIST RMF controls.
7. **Executive Dashboard** (`executive_dashboard`): A roll-up summary of organizational AI risk.
8. **Brand Profile** (`brand_profile`): External-facing transparency report.
9. **Signoff History** (`signoff_history`): Chain of custody for model approvals.

## How to Generate a Report

Reports can be generated directly through the **Enhanced Dashboard** or programmatically via the **WhiteBoxXAI Python SDK**.

### Via the Dashboard
1. Navigate to **Audit & Explanation Reports**.
2. Select **Generate New Report**.
3. Choose the desired report type from the dropdown.
4. Select the target model (or leave blank for org-wide reports).
5. Click **Download** (PDF, HTML, or DOCX formats available).

### Via the Python SDK
```python
from whiteboxxai.client import WhiteBoxXAI

client = WhiteBoxXAI()

# Generate a PDF Fairness Audit for a specific model
response = client.reports.generate(
    report_type_id="fairness_audit",
    model_id="mod_123456",
    format="pdf"
)

print(f"Report generated! Download URL: {response['url']}")
```
