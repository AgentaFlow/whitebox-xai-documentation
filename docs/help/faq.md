---
description: Answers to common questions about WhiteBoxXAI — what it does and doesn't do, plans and Governance Events, supported models and LLM providers, data handling, compliance frameworks, explanations, drift, fairness, alerts, reports, governance votes, the SDK, API and MCP server.
---

# FAQ

## About WhiteBoxXAI

### What does WhiteBoxXAI do?

It turns how your AI systems actually behave into evidence a compliance team can use. It monitors
models, LLM applications and agents; computes drift, explanations and fairness audits from real data;
tracks risks, review decisions and control evidence; and produces reports for auditors, regulators and
boards. Every number has a **How was this computed?** note saying where it came from.

### What doesn't it do?

- **It doesn't certify compliance.** It records evidence and shows gaps; your organization, auditor or
  regulator decides what the evidence proves.
- **It doesn't call your models or LLM providers** to generate scores. Evaluations are deterministic
  checks of outputs you've logged, and nothing uses another model as a judge.
- **It doesn't invent numbers.** With too little data, it says a result can't be computed yet.
- **It doesn't cover frameworks beyond five**: ISO/IEC 42001, the EU AI Act, GDPR, CCPA and NIST AI RMF.

### Who is it for?

Compliance, risk and governance teams who need defensible evidence about AI systems, and the data
scientists and engineers who run those systems. The dashboard's [lenses](../user-guide/index.md#lenses-view-as)
present the same evidence for a compliance officer, a data scientist, an executive or an auditor.

### Is it cloud or on-premise?

Both. **Free** runs on the shared WhiteBoxXAI cloud at [whiteboxxai.com](https://whiteboxxai.com).
**Business Cloud** is your own dedicated tenant in Microsoft Azure at `<your-org>.app.whiteboxxai.com`.
**Enterprise Edition** runs on your own infrastructure, including air-gapped networks. See
[Plans & Limits](../get-started/plans.md).

## Plans and usage

### How much does it cost?

Free is $0. Business Cloud is $599/month or $5,990/year. Enterprise Edition is a lifetime license with
yearly maintenance and support. Every plan has the same governance features. See
[Plans & Limits](../get-started/plans.md).

### What's a Governance Event?

One unit of recorded AI activity: a logged prediction, an LLM call, a RAG retrieval, a safety scan, an
explanation, a fairness audit, a drift check or report, or an evaluation run. A multi-agent workflow is
one event in total. Dashboard use, reads, configuration and reports are free. See
[Governance Events](../get-started/governance-events.md).

### What's a Governed AI System?

A model, LLM app or agent system you register, counted by name. New versions don't count, and archived
and sample-data systems don't either. Free includes 1, Business Cloud up to 10, Enterprise Edition
unlimited.

### What happens when I reach my limit?

On Free, recording pauses with `429 QuotaExceeded` until next month; everything else keeps working.
On Business Cloud, recording continues and extra events are billed at $5 per 10,000. Registering a
system beyond your plan's limit returns `403`.

### Is there a free trial?

The Free plan never expires and needs no credit card. Demo accounts with sample data are also
available on request, and you can load sample data into your own account from **Settings → Sample data
& tour**.

### How do I upgrade?

**Settings → Subscription & usage**, or [contact us](mailto:support@whiteboxxai.com) to set up Business
Cloud or Enterprise Edition.

## Accounts and security

### How do I sign up?

At [whiteboxxai.com](https://whiteboxxai.com/register), with Google, Microsoft, or your name, email
and a password. No company details are needed. See [Getting Started](../get-started/getting-started.md).

### I forgot my password

Choose **Forgot password** on the sign-in page and follow the emailed link.

### Should I turn on two-factor authentication?

Yes. It protects the account that holds your compliance evidence, and you need it to vote on
governance review requests. See [Two-Factor Authentication](../account/two-factor-authentication.md).

### Does WhiteBoxXAI support single sign-on?

You can sign in with Google or Microsoft accounts. SAML single sign-on and SCIM user provisioning
aren't available. Enterprise Edition runs inside your own identity boundary.

### How do I invite colleagues and set roles?

**Settings → Users & roles → Invite user**, with the role **Administrator**, **Analyst** or **Viewer**.
See [Settings](../user-guide/settings.md#users--roles).

### How do API keys work?

Administrators create them in **Settings → API keys**. A key is shown once, can be given an expiry,
and acts with the permissions of the administrator who created it. See [API Keys](../account/api-keys.md).

### How do I delete my account or export my data?

**Settings → Data & privacy**: **Download my data** (JSON) or **Delete my account**.

## Data

### What data should I log?

The inputs a model used and the outputs it returned. Don't log names, addresses or government ids; put a
non-identifying reference in `metadata` if you need to trace a decision. The SDK's
[privacy helpers](../sdk/index.md#privacy-helpers) can mask personal data first.

### Who can see my data?

Only users in your organization, according to their role. A request for another organization's record
returns `404`, exactly as if it didn't exist. Business Cloud runs in a dedicated tenant, and Enterprise
Edition in your own environment.

### Is data encrypted?

Yes, in transit (TLS) and at rest.

### Do I have to log every prediction?

No. For high-volume models, log a representative sample with `ModelMonitor(sampling_rate=...)`. Drift
and fairness results are computed from what you log, so sample consistently.

### Can I add actual outcomes to predictions later?

Not through an update to the logged prediction. Fairness audits take the true outcomes as input when
you run them, and performance metrics you compute yourself can be reported with `client.metrics.create`.

## Models and frameworks

### Which models can I monitor?

Anything you can call from Python, plus anything that can call a REST API. There are ready-made
integrations for scikit-learn, XGBoost, LightGBM, PyTorch, TensorFlow/Keras, Hugging Face Transformers,
LangChain/LangGraph and CrewAI. See [Python SDK](../sdk/index.md).

### Which LLM providers are supported?

All of them. LLM calls are logged with a free-text `provider`, so OpenAI, Anthropic Claude, Google Gemini,
xAI Grok, Mistral, AWS Bedrock, Azure OpenAI, Cohere, open models and local models all work. Cost is
calculated for model names WhiteBoxXAI has prices for (current OpenAI, Anthropic, Gemini and Azure OpenAI
models) and left blank otherwise. See [LLM monitoring](../sdk/llm-monitoring.md).

### Is there an SDK for languages other than Python?

No. Use the [REST API](../sdk/api-reference.md) from any language, or the [MCP server](../integrations/mcp.md)
from AI assistants.

### Can an AI agent use WhiteBoxXAI?

Yes, through the [MCP server](../integrations/mcp.md) (197 tools) or the SDK. Give it the
[agent skills](../agent-resources/skill-libraries.md), which tell it to report results honestly and
never vote on a person's behalf.

## Explanations, drift and fairness

### Which explanation methods are there?

SHAP (kernel and tree), LIME (tabular) and DiCE counterfactuals. Explanations run the model itself, so
the model file must be stored with WhiteBoxXAI through
[GitHub registration](../integrations/github.md#configuration-file) with `storage_mode: full`. See
[Assurance](../user-guide/assurance.md#explainability).

### How is drift detected?

Each feature in a recent window of predictions is compared with the baseline: Kolmogorov–Smirnov and
PSI for continuous features, chi-squared and Jensen–Shannon divergence for categorical ones, plus
concept-drift methods. Sensitivity presets set the thresholds. See
[Drift Detection](../user-guide/features.md#drift-detection).

### What does a fairness audit need?

True outcomes, predicted outcomes, and the group each record belongs to, which you supply.
WhiteBoxXAI doesn't infer protected attributes. See [Assurance → Fairness](../user-guide/assurance.md#fairness).

### Are LLM answers checked for safety?

Yes, if you send them to `client.safety.analyze`: toxicity categories, 12 types of personal data, and
harmful content. Jailbreak and prompt-injection detection aren't part of WhiteBoxXAI.

## Alerts

### When do alert rules fire?

When metrics are logged, drift reports are created or bias audits complete, active rules for that model
are evaluated, and any rule whose conditions are met opens an alert in
[Needs Attention](../user-guide/needs-attention.md). See
[When rules are evaluated](../user-guide/needs-attention.md#when-rules-are-evaluated).

### Which notification channels are there?

Slack, email, webhook, SMS and PagerDuty, set up by an administrator in **Settings → Integrations**.

## Reports and evidence

### What reports can I generate?

Twelve report types, including a Quarterly Board Pack, an ISO/IEC 42001 Statement of Applicability, EU AI
Act Annex IV documentation, a GDPR DPIA record, a CCPA ADMT risk assessment, a NIST AI RMF profile, model
cards, fairness and drift reports, evidence packs and an auditor traceability matrix, as PDF, Word or
HTML. See [Reports](../user-guide/reports.md).

### How do I give a report to an external auditor?

**Share with an auditor** sends a link that only works after the recipient verifies a code sent to their
email. Links expire, carry a watermark, and can be revoked. See
[Share a report with an auditor](../user-guide/reports.md#share-a-report-with-an-auditor).

### Can I schedule reports?

Yes, daily, weekly, every 30 or 90 days, or on a custom schedule. See
[Reports → Schedules](../user-guide/reports.md#schedules).

## Governance

### Can an AI assistant or pipeline approve a model?

No. Votes on review requests need a person's own signed-in session, two-factor authentication and a
current authenticator code, and are refused if the request changed since they opened it. Organization
API keys can't vote. Pipelines can gate deployments on policy instead; see
[Policy-as-Code & GitOps](../integrations/policy-as-code.md).

### Can the person who raised a request approve it?

No. Submitters can't vote on their own requests, and that's enforced on every path, including the API.

### Can a recorded decision be changed?

No. The decision archive is immutable. A mistake is corrected by recording a correction that supersedes
the original, with both kept. See [Reviews & Decisions](../user-guide/governance.md).

## Getting help

See [Troubleshooting](troubleshooting.md) for error messages, and [Getting support](support.md) to
contact us: **Support** in the dashboard sidebar, or
**[support@whiteboxxai.com](mailto:support@whiteboxxai.com)**.
