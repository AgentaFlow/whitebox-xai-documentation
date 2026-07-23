---
hide:
  - navigation
  - toc
---

<div class="wbx-hero" markdown>

# Understand *why* your AI makes decisions

<p class="wbx-tagline">
WhiteBoxXAI is the AI observability, explainability, and governance platform that helps you
monitor production models, explain their predictions, catch drift and bias early, and stay
compliant — with confidence.
</p>

[Get started](get-started/getting-started.md){ .md-button .md-button--primary }
[Install the SDK](sdk/index.md){ .md-button }
[Browse the User Guide](user-guide/index.md){ .md-button }

</div>

---

## Start here

<div class="grid cards" markdown>

-   :material-rocket-launch: **[Getting Started](get-started/getting-started.md)**

    ---

    Create your account, install the SDK, register your first model, and see monitoring in
    action — end to end.

-   :material-book-open-page-variant: **[User Guide](user-guide/index.md)**

    ---

    A tour of the platform: dashboards, explanations, drift detection, bias auditing, LLM
    monitoring, alerts, and reports.

-   :material-language-python: **[Python SDK](sdk/index.md)**

    ---

    Instrument your models in a few lines. Decorators, async logging, offline buffering, and
    multi-agent monitoring included.

-   :material-puzzle: **[Integrations](integrations/examples.md)**

    ---

    Native support for scikit-learn, PyTorch, TensorFlow, Hugging Face, LangChain, XGBoost,
    LightGBM, GitHub, and n8n.

</div>

## What you can do with WhiteBoxXAI

<div class="grid cards" markdown>

-   :material-chart-line: **Monitor in real time**

    ---

    Track production models for performance degradation, data drift, and concept drift — with
    proactive alerts before problems reach your users.

-   :material-lightbulb-on: **Explain every decision**

    ---

    Generate human-readable explanations with SHAP and LIME, so you can answer *why* a model
    made a given prediction.

-   :material-scale-balance: **Audit for fairness**

    ---

    Run bias and fairness audits and produce the evidence your stakeholders and regulators
    expect.

-   :material-gavel: **Govern with confidence**

    ---

    Multi-party [governance review boards](user-guide/governance.md), automated periodic
    reviews, and a searchable decision archive — aligned with ISO 42001 and the EU AI Act.

</div>

## Install the SDK

```bash
pip install whitebox-xai-sdk
```

```python
from whiteboxxai import WhiteBoxXAI

client = WhiteBoxXAI(api_key="your-api-key")
```

Then follow the [Getting Started guide](get-started/getting-started.md) to register a model
and log your first predictions.

## Need help?

- Browse the [FAQ](help/faq.md) and [Troubleshooting guide](help/troubleshooting.md)
- Explore hands-on [workshops, demos, and case studies](learn/index.md)
- Email us at [support@whiteboxxai.com](mailto:support@whiteboxxai.com)
