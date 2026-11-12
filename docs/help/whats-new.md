---
description: What's new in WhiteBoxXAI since August 2026 — the Enhanced Dashboard, Governance Events pricing, Report Studio, evaluations and prompt versions, SDK 1.3.0, MCP server 0.3.0, agent skills, authenticator-code voting, automatic alert rules, Policy-as-Code and more.
---

# What's new

Changes since the previous documentation update (27 August 2026), newest areas first.

## November 2026

### OpenTelemetry, Portkey, Kong and Azure API Management gateways

**Settings → LLM gateway** now has tabs for four more sources besides LiteLLM, each a separate connection
with its own token, rules and health. See [LLM gateways](../integrations/gateways.md).

- **OpenTelemetry (OTLP)**: traces from a Collector or straight from an SDK, over HTTP as JSON or Protobuf.
  Only GenAI spans are kept, and the others cost nothing
- **Portkey**: Portkey's OpenTelemetry log export, with Portkey metadata and workspaces used to file calls
- **Kong AI Gateway**: the `http-log` plugin alongside `ai-proxy`, with calls filed by Kong consumer
- **Azure API Management**: an outbound policy that records model, tokens and timing per call, with calls
  filed by APIM product and subscription
- The **Integrations** card now shows whichever connected gateway needs attention most

### LiteLLM gateway

Connect a LiteLLM gateway once and every call through it is recorded in WhiteBoxXAI and filed under the AI
system it belongs to, with no change to your teams' code. See [LiteLLM gateway](../integrations/litellm.md).

- **Settings → LLM gateway** to connect, with a token shown once and a configuration block to paste into LiteLLM
- A verdict in words on whether it is working, including when **calls are being lost** because a batch was
  refused and LiteLLM will not resend it
- **Redact personal data**, **Metadata only** or **Store everything as sent**, with the limits of pattern
  redaction stated beside the choice
- **Unregistered traffic**: every team, key, model and tag label seen in the last 30 days that no rule claims,
  each assignable to an AI system; assigning also files the last 30 days of matching calls. **Add rule**
  writes a rule for a label that hasn't appeared yet.
- A card on **Settings → Integrations** with the same verdict

## October 2026

### Enhanced Dashboard

A redesigned dashboard is now the default for every account, organized around the questions a risk team
gets asked. See [The Enhanced Dashboard](../user-guide/index.md).

- **Overview** with posture tiles, framework readiness as "N of M controls evidenced", trends, value and
  impact, a five-step **readiness setup** wizard, and **presentation mode** for board meetings
- **Needs Attention**: alerts, risks, reviews and deadlines in one queue, worst first
- **Model Register** with a **dossier** per system: verdicts you can mark reviewed, signals, model card,
  control coverage, trust score history and audit trail
- **Assurance**: explainability (now including **DiCE counterfactuals**), fairness and drift, each with a
  plain-language verdict
- **Observability**: performance, **LLM & RAG audit trails**, agents and evaluations
- **Compliance & Evidence**: framework detail for all five frameworks, control applicability decisions,
  evidence filing and approval, **Collect system evidence**, evidence packs, a compliance calendar and an
  audit trail with an integrity check
- **Reviews & Decisions**: requests, boards, the decision archive and RACI in one place
- **Lenses** ("View as"), a **⌘K** command palette, keyboard shortcuts and saved views
- A **Support** page with an in-app request form

The Classic layout remains available for now from the avatar menu.

### Compliance

- Control catalogs for **ISO/IEC 42001** (63 controls), the **EU AI Act** (28, following the Digital AI
  Omnibus amendments), **GDPR** (25), **CCPA** (22) and **NIST AI RMF** (19 categories). See
  [Compliance frameworks](../account/ai-regulations.md).
- **ISO/IEC 42001 registers**: record AIMS policies with an approval step, and measurable AI objectives,
  from the dashboard.
- **NIST AI RMF** detail by core function, with open risks per function.
- **Access reviews** in **Settings → Security posture**: keep or revoke each account, with notes, recorded
  as attestations.
- **Model card refresh** from the dossier, pulling the latest metrics, drift and bias results.
- New risk register categories: **hallucination** and **privacy**. Changing severity thresholds now
  re-rates every risk.

### Reports

The **Report Studio** replaces the old report engine: 12 report types in PDF, Word or HTML, templates with
writing style and glossary, multiple **brand profiles**, **Preparer / Reviewer / Approver sign-off**,
**Verify a file**, auditor share links with presets and watermarks, and schedules that send a secure link.
See [Reports](../user-guide/reports.md).

### Governance

- Votes and finalizations need a current **authenticator code**, and are refused if the request changed
  since the voter opened it. Organization API keys can't vote.
- Review requests take **attachments** and typed discussion comments.

### Evaluations and prompts

- **Golden datasets**, deterministic scoring (exact match, required phrases, rule-based checks), and a
  **regression check** between prompt versions. No LLM-as-a-judge.
- **Governed prompt versions** with a design → verify → deploy → retired lifecycle, rollback, and links
  from each LLM call to the version that produced it.

See [Evaluations & Prompt Versions](../user-guide/evaluations.md).

### Alerts

Alert rules are now **evaluated automatically** when metrics are logged, drift reports are created and
bias audits complete. See [Needs Attention](../user-guide/needs-attention.md#when-rules-are-evaluated).

### Policy-as-Code & GitOps

Sync a `whitebox.yaml` policy from your repository and evaluate a model against it to warn or block a
deployment from CI. The [WhiteBoxXAI Governance Action](https://github.com/AgentaFlow/whitebox-governance-action)
(`uses: AgentaFlow/whitebox-governance-action@v1`) runs both steps in GitHub Actions. Sync now refuses
an action other than `warn` or `block`, and the old `agent_ticket` / `human_review` defaults are
evaluated as `warn`. See [Policy-as-Code & GitOps](../integrations/policy-as-code.md).

### Pricing

Plans are now sized in **Governed AI Systems** and **Governance Events** instead of API calls. Free
includes 1 system and 2,500 events a month; Business Cloud up to 10 systems and 100,000 events, with
overage at $5 per 10,000. Dashboard use, reads, configuration and reports are free. See
[Plans & Limits](../get-started/plans.md) and [Governance Events](../get-started/governance-events.md).

### Python SDK 1.3.0

- `governance_run()` to group a run's records into shared Governance Events
- `client.prompts`, `client.compliance` and `client.reports`
- Model version comparison (`compare_versions`, `version_changelog`, `list_git_versions`)
- DiCE counterfactual parameters on explanations
- `client.governance.get_review_request` and `submit_decision`
- Clearer rate-limit handling: long `Retry-After` waits are raised rather than slept through

See [Python SDK](../sdk/index.md) and [LLM monitoring](../sdk/llm-monitoring.md).

### MCP server 0.3.0

- **197 tools** across 18 domains, including prompts, compliance, model diff and alert triage
- **MCP Apps** interactive cards for explanations, alert triage, governance dossiers, ISO/IEC 42001 status
  and model version diffs
- API-key-only local authentication, and a hosted endpoint at `https://api.whiteboxxai.com/mcp/`
- Governance voting from the dossier card, with a person's own session and authenticator code

See [MCP Server](../integrations/mcp.md).

### For AI agents

Seven public [agent skills](../agent-resources/skill-libraries.md), a compliance-assistant prompt, and a
[`llms.txt`](https://docs.whiteboxxai.com/llms.txt) index of these docs.
