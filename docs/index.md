---
description: WhiteBoxXAI documentation — turn how your AI systems actually behave into audit evidence for ISO/IEC 42001, the EU AI Act, GDPR, CCPA and NIST AI RMF, using the Enhanced Dashboard, the Python SDK, the REST API or the MCP server.
hide:
  - navigation
  - toc
---

<div class="wbx-hero" markdown>

# Audit evidence and explanations you don't have to translate yourself

<p class="wbx-tagline">
WhiteBoxXAI packages what your AI actually did (SHAP, LIME and counterfactual explanations, bias
audits, drift history, LLM audit trails, governance decisions) into evidence you can hand to an auditor
or a board, with every number traceable to the data that produced it. It's also the monitoring platform
your engineering team uses to produce that evidence in the first place.
</p>

[Get started](get-started/getting-started.md){ .md-button .md-button--primary }
[Generate a report](user-guide/reports.md){ .md-button }
[Install the SDK](sdk/index.md){ .md-button }

</div>

---

## Start here

<div class="grid cards" markdown>

-   :material-rocket-launch: **[Getting Started](get-started/getting-started.md)**

    ---

    Create an account, run the readiness setup, connect a model, and generate your first report.

-   :material-view-dashboard: **[The Enhanced Dashboard](user-guide/index.md)**

    ---

    Overview, Needs Attention, Model Register, Assurance, Observability, Compliance & Evidence,
    Reviews & Decisions and the Risk Register: what each shows and how to use it.

-   :material-file-certificate: **[Reports](user-guide/reports.md)**

    ---

    Board packs, ISO/IEC 42001 Statements of Applicability, EU AI Act Annex IV documentation, model
    cards and evidence packs, signed off and shared with auditors through verified links.

-   :material-language-python: **[Python SDK](sdk/index.md)**

    ---

    Register models, log predictions and LLM calls from any provider, and group runs into Governance
    Events in a few lines.

-   :material-robot: **[MCP & AI agents](agent-resources/index.md)**

    ---

    Give Claude, Copilot, Cursor or your own agent 197 tools over your WhiteBoxXAI data, with agent
    skills that keep it honest.

-   :material-school: **[Tutorials](learn/index.md)**

    ---

    Explore with sample data, take a model to an evidence pack, govern an LLM app, or connect an
    assistant over MCP.

</div>

## What you can do with WhiteBoxXAI

<div class="grid cards" markdown>

-   :material-lightbulb-on: **Explain every decision**

    ---

    SHAP, LIME and DiCE counterfactuals computed from your model, so you can answer *why* a
    prediction was made and *what would have changed it*.

-   :material-scale-balance: **Audit for fairness**

    ---

    Fairness audits across the groups you define, with per-group comparisons and the
    [evidence](user-guide/reports.md) regulators expect, not a summary of one.

-   :material-clipboard-check: **Show compliance readiness**

    ---

    Control catalogs for ISO/IEC 42001, the EU AI Act, GDPR, CCPA and NIST AI RMF, with your
    applicability decisions and approved evidence counted control by control.
    [Compliance frameworks](account/ai-regulations.md)

-   :material-gavel: **Govern with confidence**

    ---

    [Review boards](user-guide/governance.md) with votes signed by an authenticator code, separation
    of duties, and an immutable decision archive.

-   :material-shield-alert: **Track AI risk**

    ---

    An [AI Risk Register](user-guide/risk-register.md) with owners, likelihood × impact scoring and a
    full audit trail, and a [Trust Score](user-guide/trust-score.md) per model a board can read.

-   :material-chart-line: **Monitor LLMs, agents and models**

    ---

    Drift, performance, LLM and RAG audit trails, safety scans, prompt versions and multi-agent
    workflows, with alert rules evaluated as new evidence arrives.

</div>

## Install the SDK

```bash
pip install whitebox-xai-sdk
```

```python
import os

from whiteboxxai import WhiteBoxXAI

client = WhiteBoxXAI(api_key=os.environ["WHITEBOX_API_KEY"])
```

Then follow [Getting Started](get-started/getting-started.md) to register a model and log your first
predictions. AI agents can start from [`llms.txt`](https://docs.whiteboxxai.com/llms.txt).

## Need help?

- [What's new](help/whats-new.md), the [FAQ](help/faq.md) and [Troubleshooting](help/troubleshooting.md)
- [Getting support](help/support.md), or email [support@whiteboxxai.com](mailto:support@whiteboxxai.com)
