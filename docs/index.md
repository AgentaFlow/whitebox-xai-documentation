---
hide:
  - navigation
  - toc
---

<div class="wbx-hero" markdown>

# Audit evidence and explanations you don't have to translate yourself

<p class="wbx-tagline">
WhiteBox XAI packages what your AI actually did — SHAP and LIME explanations, bias audits,
drift history, governance decisions — into reports you can hand to an auditor or a board
without building a slide deck at midnight. It's also the observability and monitoring
platform your engineering team uses to produce that evidence in the first place.
</p>

[Generate an Audit Report](/user-guide/reports/){ .md-button .md-button--primary }
[Get started](/get-started/getting-started/){ .md-button }
[Install the SDK](/sdk/){ .md-button }

</div>

---

## Start here

<div class="grid cards" markdown>

-   :material-file-certificate: **[Audit & Explanation Reports](/user-guide/reports/)**

    ---

    Generate bias-audit, compliance, and explainability reports from the dashboard — no code
    required — with every number traceable back to real computed data.

-   :material-rocket-launch: **[Getting Started](/get-started/getting-started/)**

    ---

    Create your account, install the SDK, register your first model, and see monitoring in
    action — end to end.

-   :material-book-open-page-variant: **[User Guide](/user-guide/)**

    ---

    A tour of the platform: dashboards, explanations, drift detection, bias auditing, LLM
    monitoring, alerts, and reports.

-   :material-language-python: **[Python SDK](/sdk/)**

    ---

    Instrument your models in a few lines. Decorators, async logging, offline buffering, and
    multi-agent monitoring included.

-   :material-puzzle: **[Integrations](/integrations/examples/)**

    ---

    Native support for scikit-learn, PyTorch, TensorFlow, Hugging Face, LangChain, XGBoost,
    LightGBM, GitHub, n8n — and an [MCP server](/integrations/mcp/) for any AI client.

</div>

## What you can do with WhiteBox XAI

<div class="grid cards" markdown>

-   :material-lightbulb-on: **Explain every decision**

    ---

    Generate human-readable explanations with SHAP and LIME, so you can answer *why* a model
    made a given prediction — and hand the explanation to someone who wasn't in the room.

-   :material-scale-balance: **Audit for fairness**

    ---

    Run bias and fairness audits and produce the [evidence](/user-guide/reports/) your
    stakeholders and regulators expect — not a summary of one.

-   :material-gavel: **Govern with confidence**

    ---

    Multi-party [governance review boards](/user-guide/governance/), automated periodic
    reviews, and an immutable decision archive — aligned with ISO 42001 and the EU AI Act.

-   :material-shield-alert: **Track AI risk**

    ---

    A structured [AI Risk Register](/user-guide/risk-register/) with owners, likelihood ×
    impact scoring, and a full audit trail — the artifact ISO 42001 and the EU AI Act ask for.

-   :material-speedometer: **Report one number**

    ---

    A [Trust Score](/user-guide/trust-score/) per model, combining fairness, drift, and
    explainability into a single 0–100 index your board can actually read.

-   :material-chart-line: **Monitor in real time**

    ---

    Track production models for performance degradation, data drift, and concept drift — with
    proactive alerts before problems reach your users.

</div>

## Install the SDK

```bash
pip install whitebox-xai-sdk
```

```python
from whiteboxxai import WhiteBoxXAI

client = WhiteBoxXAI(api_key="your-api-key")
```

Then follow the [Getting Started guide](/get-started/getting-started/) to register a model
and log your first predictions.

## Need help?

- Browse the [FAQ](/help/faq/) and [Troubleshooting guide](/help/troubleshooting/)
- Explore hands-on [workshops, demos, and case studies](/learn/)
- Email us at [support@whiteboxxai.com](mailto:support@whiteboxxai.com)
