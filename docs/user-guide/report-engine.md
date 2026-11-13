---
description: Reference for every WhiteBoxXAI report type — id, scope, the data each one needs before it can be generated, formats, and the SDK and REST calls to list, preview and generate them.
---

# Report types reference

This page lists every report type the [Report Studio](reports.md) can generate, with the id to use
from code and the data each one needs. For how to customize, brand, sign off, share and schedule
reports, see [Reports](reports.md).

All report types generate as **PDF**, **Word (DOCX)** or **HTML**.

## Report types

| Id | Name | Scope | Needs before it can be generated |
| --- | --- | --- | --- |
| `quarterly_board_pack` | Quarterly Board Pack | Organization | At least one active model |
| `iso42001_statement_of_applicability` | ISO/IEC 42001 Statement of Applicability | Framework: ISO/IEC 42001 | ISO/IEC 42001 applicability decisions |
| `eu_ai_act_annex_iv_technical_documentation` | EU AI Act Annex IV Technical Documentation | One AI system | An EU AI Act risk classification of **high** for that system |
| `eu_ai_act_conformity_checklist` | EU AI Act Conformity Checklist | Framework: EU AI Act | EU AI Act applicability decisions |
| `gdpr_dpia_automated_decision_record` | GDPR Art. 35 DPIA / Art. 22 Automated-Decision Record | Framework: GDPR | Article 30 processing records or automated-decision requests |
| `ccpa_admt_risk_assessment` | CCPA ADMT Risk Assessment | Framework: CCPA | Applicability decisions for the CCPA ADMT controls |
| `nist_ai_rmf_profile` | NIST AI RMF Profile | Framework: NIST AI RMF | NIST AI RMF applicability decisions |
| `model_card` | Model Card | One AI system | An authored model card for that system |
| `fairness_audit` | Fairness Audit | One AI system | At least one bias audit for that system |
| `drift_report` | Drift Report | One AI system | At least one drift report for that system |
| `evidence_pack` | Evidence Pack | Organization | At least one active model |
| `auditor_traceability_matrix` | Auditor Traceability Matrix | Organization | Control applicability decisions for at least one framework |

Applicability decisions are made under **Compliance & Evidence → Controls** (see
[Compliance & Evidence](compliance-evidence.md#controls)). When a report type's needs aren't met, it
reports **Needs data** with the specific reason. It never produces a partial document filled with
placeholders.

## Generate from code

=== "Python SDK"

    ```python
    types = client.reports.list()
    for t in types:
        print(t["id"], t["name"])

    html = client.reports.preview(report_type_id="drift_report", model_id=model_id)
    export = client.reports.generate(report_type_id="drift_report", model_id=model_id, format="pdf")
    ```

=== "REST"

    ```bash
    # List report types, with availability for your organization
    curl https://api.whiteboxxai.com/api/v1/report-types \
      -H "Authorization: Bearer $WHITEBOX_API_KEY"

    # Preview as HTML
    curl -X POST "https://api.whiteboxxai.com/api/v1/report-types/drift_report/preview?model_id=<model-id>"       -H "Authorization: Bearer $WHITEBOX_API_KEY"

    # Generate; the response is an export record with an id
    curl -X POST "https://api.whiteboxxai.com/api/v1/report-types/drift_report/generate?format=pdf&model_id=<model-id>"       -H "Authorization: Bearer $WHITEBOX_API_KEY"
    ```

    `format` (`pdf`, `docx` or `html`), `model_id`, `brand_profile_id` and `template_id` are query
    parameters. To override a template's writing style for one run, send the settings as the JSON
    body.

The generated report is an **export**: download it from
`GET /api/v1/export/exports/{export_id}/download`, and share or sign it off as described in
[Reports](reports.md#from-code). With SDK 1.4.0 or later, `client.reports.download(result,
path="report.pdf")` does the download for you; see [Reports](reports.md#from-code).

Every type shares the same layout: a document-control table, a findings summary, "What this section
shows an auditor" notes and (in HTML and PDF) a contents list. See
[How every report is laid out](reports.md#how-every-report-is-laid-out).

Pass `model_id` only for report types scoped to one AI system. Organization- and framework-scoped
types ignore it.

## Generate from an AI assistant

An AI assistant generates reports through the Python SDK, using the calls above. The
[MCP server](../integrations/mcp.md) doesn't have report-generation tools; over MCP, an assistant can
create and read **drift reports** and read the evidence a report draws on. Have the assistant take
report type ids from `client.reports.list()` rather than guessing, and give you the download link and
generation date. The [compliance-evidence skill](../agent-resources/skill-libraries.md) covers this.
