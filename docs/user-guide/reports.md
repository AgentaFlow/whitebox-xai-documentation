---
description: WhiteBoxXAI Reports — 12 audit-ready report types (board pack, ISO/IEC 42001 SoA, EU AI Act Annex IV, GDPR DPIA, CCPA ADMT, NIST AI RMF, model card, fairness, drift, evidence pack, traceability matrix), branded templates, sign-off, verified auditor share links and schedules.
---

# Reports

You have a deadline: an EU AI Act milestone, an ISO/IEC 42001 audit, a customer's vendor
questionnaire, a board meeting. You need a document you can hand over *without* translating it
yourself first. The **Report Studio** produces it from the evidence WhiteBoxXAI already holds, in
PDF, Word or HTML, with your branding, sign-off and a verified way to share it.

It's under **Compliance & Evidence → Reports**, with three sub-tabs: **Report types**, **History**
and **Schedules**.

## Why you can trust what's in a report

- **Every number traces to a computation.** Explanations are live SHAP, LIME or DiCE output, and
  fairness and drift sections come from recorded audits and reports you can open by id.
- **Nothing is templated filler.** A report type that lacks the data it needs says **Needs data**,
  with the reason, rather than generating a document with placeholder text.
- **Governance records are enforced, not asserted.** Approvals, votes, control decisions and
  justifications in a report are the ones recorded in the platform, with who and when.
- **Every issued file can be checked later.** **Verify a file** confirms that a document someone
  hands back is byte-for-byte the one WhiteBoxXAI generated.

For how SHAP, LIME and fairness metrics are computed, see [Assurance](assurance.md) and the
[Feature reference](features.md).

## Report types

| Report type | Scope | What it contains |
| --- | --- | --- |
| **Quarterly Board Pack** | Whole organization | Trust, risk and compliance posture across every monitored model, in plain governance language, including a **High-Risk AI Systems** section |
| **ISO/IEC 42001 Statement of Applicability** | ISO/IEC 42001 | Every control, your in/out-of-scope decision and justification, and a traceability matrix to the evidence for each in-scope control |
| **EU AI Act Annex IV Technical Documentation** | One AI system | The Annex IV technical documentation for a high-risk system: intended purpose, design, data governance and risk management measures |
| **EU AI Act Conformity Checklist** | EU AI Act | Your applicability decisions against the Act's conformity requirements, with justifications for anything out of scope |
| **GDPR Art. 35 DPIA / Art. 22 Automated-Decision Record** | GDPR | Article 30 processing records and Article 22 automated-decision rights requests, in one accountability record |
| **CCPA ADMT Risk Assessment** | CCPA | A risk assessment for automated decision-making technology under CCPA/CPRA |
| **NIST AI RMF Profile** | NIST AI RMF | Your applicability decisions mapped to the framework's functions and categories |
| **Model Card** | One AI system | Intended use, training data, performance and fairness evaluation results |
| **Fairness Audit** | One AI system | The most recent bias audit: fairness scores, per-group comparisons and flagged severity |
| **Drift Report** | One AI system | The most recent drift analysis, with the baseline and comparison periods it was measured against |
| **Evidence Pack** | Whole organization or one system | Trust, risk and compliance evidence bundled for an auditor |
| **Auditor Traceability Matrix** | Whole organization | Every in-scope control across every adopted framework, mapped to the evidence an auditor would check |

### Status

Each report type card shows whether it can be generated now:

- **Ready**
- **Needs data**, with the reason, such as *"No bias audit has been run yet for credit_scoring"*,
  *"No ISO/IEC 42001 applicability decisions are recorded yet"*, or *"credit_scoring is classified
  limited risk; Annex IV technical documentation applies to high-risk systems"*
- **Not available yet**

### Generate a report

1. Choose a report type, and the AI system if its scope is one system.
2. Choose a format: **PDF**, **Word (DOCX)** or **HTML**.
3. Click **Generate**. The report is saved to **History**.

**Customize…** opens the template editor, and **Saved templates** lists your versions of that
report type, with **Edit** and **Generate**.

## Templates and writing style

The template editor has a live **Preview**. A template sets:

- **Template name**, **Format**, **Brand profile** (or **Organization default**) and **AI system**
- **Writing style**:
    - **Audience**: Compliance officer, Data scientist, Board and executives, Auditor or
      Administrator
    - **Tone**: Formal or Plain
    - **Level of detail**: Concise, Standard or Detailed
    - **Spelling**: US or UK
- **Glossary**: replace a platform term with your organization's term (**Platform says** → **Say
  instead**)
- **Sections**: include, remove and reorder sections, and **Add introduction** with placeholders
- **Default template**: use this template whenever the report type is generated

**Save template** keeps it, and **Generate** produces a report from it.

## Brand profiles

Reports carry your organization's branding from a **brand profile**. You can keep several, with one
set as the **Default**. Administrators edit them in **Settings → Brand profiles**; everyone else can
view them. A profile sets:

- **Colours**, with a contrast check
- **Font**: Inter, Arial, Helvetica, Georgia or Times New Roman
- **Cover layout**: Classic, Modern or Minimal
- **Header and footer text**, **Confidentiality marking** and **Disclaimer**
- **Watermark**: None, DRAFT or CONFIDENTIAL
- **Page size**: A4 or US Letter
- **Logo**

Margin text and the watermark repeat on every page of a PDF.

## History, sign-off and verification

**Report history** lists every generated report, with **Download**.

### Sign-off

A report can be signed off in three roles: **Preparer**, **Reviewer** and **Approver**. Use
**Sign as …** on a report. Each person can hold only **one** role on a given file, so the person who
prepared a report can't also approve it. Sign-offs are recorded against that exact file.

### Verify a file

**Verify a file** checks a document you've been handed against the reports WhiteBoxXAI generated. If
the file was altered after it was issued, it won't match.

## Share a report with an auditor

Sharing a report with someone outside your organization, such as an external auditor or a
customer's security team, doesn't mean emailing a PDF or creating them an account.

1. In **History**, choose **Share with an auditor** on a report.
2. Pick a preset:

    | Preset | Link expires after | Watermark |
    | --- | --- | --- |
    | **Share with auditor** | 30 days | CONFIDENTIAL |
    | **Quick preview** | 2 days | DRAFT |

3. Enter the recipient's email address and **Send share link**.

The recipient opens a page titled **Evidence report shared with you**, chooses **Email me a
verification code**, enters the 6-digit code sent to that same address, and then **Download
report**. The share list shows each share's state, such as **Waiting for them to verify**,
**Expired** or revoked, and **Revoke** stops a share immediately.

How the link is protected:

- The link is a **locator, not a credential**. Opening it reveals nothing until the recipient proves
  they control the email address.
- The download token issued after verification is scoped to that one report and expires after
  **15 minutes**.
- Revoking a share blocks any further code request or verification immediately.
- Every step (created, code requested, verified, denied, viewed, revoked) is written to the
  [audit trail](compliance-evidence.md#audit-trail).

## Schedules

**Schedules → New schedule** runs a template automatically:

- **Name**, **Template to run** and **Active**
- **How often**: **Every day**, **Every week**, **Every 30 days**, **Every 90 days**, or **Custom**
  (a cron expression, in UTC)
- **Tell these addresses when it finishes**: who is notified when a run completes
- **Also e-mail the report as a secure link** → **Send the report to**: recipients who get the report
  itself, through the same verified share link as above, never as an attachment

Each schedule shows **Next run**, **Last run**, **Runs** and **Failures**. Failed runs are retried,
and each schedule runs once per period.

## From code

Generate a report type through the SDK:

```python
types = client.reports.list()        # ids, names, scope and availability
result = client.reports.generate(report_type_id="fairness_audit", model_id=model_id, format="pdf")
html = client.reports.preview(report_type_id="fairness_audit", model_id=model_id)
```

Report type ids: `quarterly_board_pack`, `iso42001_statement_of_applicability`,
`eu_ai_act_annex_iv_technical_documentation`, `eu_ai_act_conformity_checklist`,
`gdpr_dpia_automated_decision_record`, `ccpa_admt_risk_assessment`, `nist_ai_rmf_profile`,
`model_card`, `fairness_audit`, `drift_report`, `evidence_pack`, `auditor_traceability_matrix`.
Take ids from `client.reports.list()` rather than hard-coding them.

A generated report is an **export** with an id. Download, share and sign it through the export
endpoints:

```bash
# Download
curl -L -o report.pdf https://api.whiteboxxai.com/api/v1/export/exports/{export_id}/download \
  -H "Authorization: Bearer $WHITEBOX_API_KEY"

# Share with an auditor (presets: share_with_auditor, quick_preview)
curl -X POST https://api.whiteboxxai.com/api/v1/export/exports/{export_id}/share \
  -H "Authorization: Bearer $WHITEBOX_API_KEY" -H "Content-Type: application/json" \
  -d '{"recipient_email": "auditor@example.com", "preset": "share_with_auditor"}'

# Record a sign-off: "prepared", "reviewed" or "approved"
curl -X POST https://api.whiteboxxai.com/api/v1/export/exports/{export_id}/signoffs \
  -H "Authorization: Bearer $WHITEBOX_API_KEY" -H "Content-Type: application/json" \
  -d '{"role": "reviewed"}'
```

Data exports in other formats (CSV, Excel, JSON, Markdown) for model performance, drift, bias,
explainability, LLM monitoring, the risk register and the trust score are available from
`POST /api/v1/export/exports`, with schedules under `/api/v1/export/scheduled-reports`. See the
[API Reference](../sdk/api-reference.md#reports-and-exports).

Generating reports, sharing and scheduling don't count as
[Governance Events](../get-started/governance-events.md).

## FAQ

**Is a report a snapshot or live?** A snapshot. A generated report records the evidence as it was
when generated, so it stays stable for an audit. Generate again, or schedule it, to reflect newer
data.

**What if part of a report has no data?** It says so. An empty, labelled section is more
defensible under audit than a filled-in one.

**Can an AI assistant generate reports?** Yes, through the Python SDK. The
[MCP server](../integrations/mcp.md) has no report-generation tools.
The [compliance-assistant prompt](../agent-resources/prompts.md) tells it to give you the download
link and date rather than paraphrase a report it hasn't read.
