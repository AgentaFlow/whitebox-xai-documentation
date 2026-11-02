---
description: Go from a new WhiteBoxXAI account to a monitored model with drift, explainability and an exportable evidence report — sign up, set up readiness, connect the SDK, log predictions, and generate your first report.
---

# Getting Started with WhiteBoxXAI

An auditor, a regulator or your own review board will eventually ask how a production AI system
behaves and who signed off on it. This guide gets you from a new account to a monitored model with
evidence you could hand over: real predictions logged, drift and explanations computed from them,
and a report generated from the result.

It takes about 20 minutes. Prefer to explore first? Load sample data (step 2) and skip to
[Find your way around](#5-find-your-way-around).

!!! note "Using an AI coding agent?"
    Give it this instruction and it can do steps 3 to 5 for you:
    `Read https://whiteboxxai.com/.well-known/skills/whitebox-getting-started/SKILL.md and set up WhiteBoxXAI for this repository.`
    See [Agent Skills](../agent-resources/skill-libraries.md).

## Before you start

- **Python 3.9 or later**, if you'll connect a model with the SDK
- **A model you can call**: scikit-learn, XGBoost, LightGBM, PyTorch, TensorFlow/Keras, Hugging
  Face, an LLM app, or anything else you can call from Python

No Python? You can still register models, review evidence, run the readiness setup and generate
reports entirely from the dashboard.

## 1. Create your account

1. Go to **[whiteboxxai.com](https://whiteboxxai.com)** and choose **Get Started for Free**.
2. Sign up with **Google** or **Microsoft**, or with email. The email form asks for:
    - **Full Name**
    - **Email**
    - **Password**: at least 10 characters, with at least one letter and one number
    - **Confirm Password**
3. Optionally tick the box to receive product updates. You can change this later in **Settings →
   Data & privacy**.
4. Click **Create Account**. If you're asked to verify your email, follow the link we send.

You're the **Administrator** of a new organization on the [Free plan](plans.md): one Governed AI
System and 2,500 Governance Events a month. You can invite colleagues later from **Settings → Users &
roles**.

!!! tip "Turn on two-factor authentication now"
    Two-factor authentication protects the account that will hold your compliance evidence. You'll
    also need it to **vote on governance review requests**. Go to **Settings → Profile & security**.
    See [Two-Factor Authentication](../account/two-factor-authentication.md).

## 2. Set up compliance readiness

The first time you sign in, WhiteBoxXAI opens on the **Enhanced Dashboard** and offers **Set up your
compliance readiness**. You can also start it any time from **Overview → Set up readiness**.

The setup has five steps. Each step is checked against your real data, not a checkbox you tick:

1. **Frameworks**: adopt the frameworks you're accountable to: ISO/IEC 42001, the EU AI Act, GDPR,
   CCPA, or NIST AI RMF.
2. **Audit dates**: set your next audit date for each framework. It appears on the compliance
   calendar and drives deadline reminders.
3. **Applicability**: review the default decisions about which controls apply to you.
4. **Connect the SDK**: send real data from a model (steps 3 to 5 below).
5. **Invite a reviewer**: invite a colleague who will review and approve AI changes.

Your progress shows on the Overview as **Your readiness setup: N of 5 steps done**. Analysts can
propose framework adoptions and dates, and an administrator confirms them.

!!! tip "Want to see a populated dashboard first?"
    Go to **Settings → Sample data & tour → Load sample data**. Sample models, predictions, drift
    events, bias audits and reports are added and clearly marked as sample data. They don't count
    against your plan, and **Reset sample data** removes them. WhiteBoxXAI never shows invented
    numbers on your own systems: an account with no data shows empty states.

## 3. Install the SDK and create an API key

```bash
pip install whitebox-xai-sdk
```

Optional extras install framework integrations: `whitebox-xai-sdk[sklearn]`, `[pytorch]`,
`[tensorflow]`, `[xgboost]`, `[lightgbm]`, `[huggingface]`, `[langchain]`, or `[all]`.

Create an API key in **Settings → API keys** (administrators only). Copy it when it's shown; it's
displayed once. Keys look like `wbx_live_...`. Then put it in your environment rather than in code:

=== "macOS / Linux"

    ```bash
    export WHITEBOX_API_KEY="wbx_live_..."
    ```

=== "Windows PowerShell"

    ```powershell
    $env:WHITEBOX_API_KEY = "wbx_live_..."
    ```

Check the connection:

```python
import os

from whiteboxxai import WhiteBoxXAI

client = WhiteBoxXAI(api_key=os.environ["WHITEBOX_API_KEY"])
print(client.models.list(limit=5))
```

An authentication error means the key is wrong or revoked. See [API Keys](../account/api-keys.md).

!!! note "Business Cloud"
    On Business Cloud, your organization has its own tenant. Use the API address you were given
    when the tenant was set up, by passing `base_url=` or setting `WHITEBOX_BASE_URL`.

## 4. Register a model and log predictions

Register the model once. Calling `register` again with the same name and version returns the
existing model, so it's safe to run on every start-up.

```python
model = client.models.register(
    name="credit_scoring",
    model_type="xgboost",          # xgboost, random_forest, neural_network, linear_model, llm or custom
    framework="xgboost",
    version="1.0.0",
    features=["income", "utilisation", "tenure_months"],
    baseline_metrics={"accuracy": 0.94},   # only values you actually measured; {} if none
    auto_detect_git=True,          # records the commit and branch the model came from
)
model_id = model["id"]
```

Then log what the model actually received and returned:

```python
client.predictions.log(
    model_id=model_id,
    input_data={"income": 52000, "utilisation": 0.31, "tenure_months": 18},
    output_data={"prediction": 0, "probability": 0.12},
    latency_ms=4.2,
)
```

Rather not call `log` yourself? Wrap the model and every prediction is logged for you:

```python
from whiteboxxai.integrations.sklearn import SklearnMonitor, SklearnWrapper

monitor = SklearnMonitor(client, model=clf)          # clf: your fitted estimator
monitor.register_from_model(
    name="credit_scoring",
    model_type="random_forest",
    version="1.0.0",
    features=["income", "utilisation", "tenure_months"],
    baseline_metrics={},
)
wrapped = SklearnWrapper(clf, monitor)
wrapped.predict(X)               # logged automatically
```

Integrations exist for XGBoost, LightGBM, PyTorch, TensorFlow/Keras and Hugging Face too. See
[SDK & API](../sdk/index.md). For LLM, RAG and agent apps, see [LLM monitoring](../sdk/llm-monitoring.md).

!!! warning "Log only what you need"
    Log the features the model uses. Don't log names, addresses or government ids. Put a
    non-identifying reference such as a customer id in `metadata` if you need to trace a decision.

Each logged prediction is one [Governance Event](governance-events.md). For very high-volume
models, log a representative sample, or group related records into runs.

You can also register a model without code: **Model Register → Register a system**.

## 5. Find your way around

Open the **[Enhanced Dashboard](../user-guide/index.md)**. The sidebar is grouped by what you're
trying to do:

| Area | What you'll find |
| --- | --- |
| **Overview** | Where your AI systems stand: trust score, open findings, framework readiness, and what changed |
| **Needs Attention** | Every alert, risk, review and deadline that needs a person, worst first |
| **Model Register** | Every AI system you govern, with a dossier per system |
| **Assurance** | Explainability, fairness and drift for a system |
| **Observability** | Runtime performance, LLM & RAG activity, agents and prompt evaluations |
| **Compliance & Evidence** | Frameworks, controls, evidence, reports, the compliance calendar and the audit trail |
| **Reviews & Decisions** | Review requests, boards, votes and the decision archive |
| **Risk Register** | Your AI risks, scored by likelihood and impact |

Press **⌘K** (or **Ctrl+K**) to search systems, controls, reports and reviews, and **?** to see
keyboard shortcuts. Choose **Take the product tour** from the ⌘K menu for a guided walkthrough.

## 6. Check drift and explain a prediction

Once predictions are flowing, ask for evidence computed from them:

```python
report = client.drift.create_report(model_id, window_size=1000)   # stored, citable by id
explanation = client.explanations.generate(
    model_id,
    instance={"income": 52000, "utilisation": 0.31, "tenure_months": 18},
    method="shap",
)
```

Explanations run the model itself, so they need the model file stored with WhiteBoxXAI: register
the model through [GitHub](../integrations/github.md#configuration-file) with `storage_mode: full`.
With metadata-only registration, as in step 4, drift and fairness work but explanation requests
are recorded as failed.

Or use **Assurance** in the dashboard, which shows each result with a plain-language verdict and
a **How was this computed?** note. With little or no data, WhiteBoxXAI says the result can't be
computed yet rather than showing a number. Drift needs a meaningful window of predictions to
compare against the baseline.

See [Assurance](../user-guide/assurance.md).

## 7. Generate your first report

1. Go to **Compliance & Evidence → Reports**.
2. Pick a report type: for example **Model Card**, **Drift Report** or **Quarterly Board Pack**.
   Each card says whether it's **Ready** or what data it still needs.
3. Choose PDF, Word (DOCX) or HTML and click **Generate**.

The report is saved to **History**, where you can download it, record **Preparer / Reviewer /
Approver** sign-off, or share it with an auditor through a verified, expiring link.

From code:

```python
types = client.reports.list()            # pick a report type id from this list
result = client.reports.generate(report_type_id="drift_report", model_id=model_id, format="pdf")
```

See [Reports](../user-guide/reports.md).

## Next steps

**Set up governance**

- [Compliance & Evidence](../user-guide/compliance-evidence.md): adopt frameworks, decide control
  applicability and attach evidence
- [Reviews & Decisions](../user-guide/governance.md): review boards and approval workflows
- [AI Risk Register](../user-guide/risk-register.md) and [Trust Score](../user-guide/trust-score.md)

**Instrument more systems**

- [LLM monitoring](../sdk/llm-monitoring.md): OpenAI, Anthropic, Gemini, Grok, Mistral, Bedrock and more
- [Multi-agent monitoring](../sdk/multi-agent-monitoring.md): CrewAI, LangChain/LangGraph, AutoGen, n8n
- [MCP Server](../integrations/mcp.md): let an AI assistant query and act on WhiteBoxXAI

**Reference**

- [Plans & Limits](plans.md) and [Governance Events](governance-events.md)
- [SDK & API](../sdk/index.md) and the [REST API reference](../sdk/api-reference.md)
- [Troubleshooting](../help/troubleshooting.md) and the [FAQ](../help/faq.md)

## Quick troubleshooting

**`AuthenticationError` or 401.** The key is missing, mistyped or revoked. Check that
`WHITEBOX_API_KEY` is set in the same shell that runs your code, and create a new key if needed.

**`429` with `QuotaExceeded`.** Your Free plan's monthly Governance Events are used up. See
[Governance Events](governance-events.md#when-you-reach-the-limit).

**`403` when registering a model.** All of your plan's Governed AI Systems are in use. Archive one
you no longer govern, or register a new *version* of an existing system instead.

**`422` when registering a model.** Check that `model_type` is one of `xgboost`, `random_forest`,
`neural_network`, `linear_model`, `llm` or `custom`, and that `version`, `features` and
`baseline_metrics` are present.

**Predictions don't appear.** Make sure you're looking at the model id you logged to, and call
`client.predictions.get_recent(model_id)` to confirm they arrived. If you enabled offline mode, the
SDK may still be holding them in its local queue. See [Offline Mode](../sdk/offline-mode.md).

**Lost your authenticator.** Sign in with one of your backup codes. See
[Two-Factor Authentication](../account/two-factor-authentication.md).

Still stuck? Email **[support@whiteboxxai.com](mailto:support@whiteboxxai.com)** or use **Support**
in the dashboard sidebar.
