---
description: The five compliance frameworks WhiteBoxXAI covers — ISO/IEC 42001, the EU AI Act, GDPR, CCPA and NIST AI RMF — with exactly what each control catalog contains, what it leaves out, which evidence and reports map to it, and what WhiteBoxXAI does not do.
---

# Compliance frameworks

WhiteBoxXAI covers five frameworks: **ISO/IEC 42001**, the **EU AI Act**, **GDPR**, **CCPA** and the
**NIST AI RMF**. For each, it ships a control catalog, tracks your applicability decision and evidence
for every control, and generates framework-specific reports.

What it doesn't do is just as important for an auditor to understand:

- **WhiteBoxXAI records evidence; it does not certify compliance.** It shows which controls you've
  decided apply, which have approved evidence, and where the gaps are. Whether that evidence is
  sufficient is a judgement for your organization, your auditor or your regulator.
- **Readiness is a count, not a percentage.** The dashboard says "18 of 25 controls evidenced", never
  "72% compliant", because a percentage invites a conclusion the evidence doesn't support.
- **Control titles are paraphrased summaries**, not the operative text of the standard or regulation.
  Use the official text for audit-grade wording.
- **Other frameworks aren't covered.** WhiteBoxXAI doesn't map evidence to HIPAA, SOC 2, US state AI
  laws or any framework not listed here.

Where to work with them: [Compliance & Evidence](../user-guide/compliance-evidence.md) for controls and
evidence, and [Reports](../user-guide/reports.md) for framework documents.

## At a glance

| Framework | Catalog | What's covered | Framework reports |
| --- | --- | --- | --- |
| **ISO/IEC 42001:2023** | 63 controls, 16 groups | Clauses 4–10 (AI management system requirements) and all 38 Annex A controls | Statement of Applicability |
| **EU AI Act** (Regulation (EU) 2024/1689) | 28 controls, 10 groups | Provider and deployer obligations per AI system | Annex IV Technical Documentation, Conformity Checklist |
| **GDPR** (Regulation (EU) 2016/679) | 25 controls, 4 groups | Articles most relevant to AI on personal data | Art. 35 DPIA / Art. 22 Automated-Decision Record |
| **CCPA**, as amended by the CPRA | 22 controls, 5 groups | Automated decision-making technology (ADMT), risk assessments and consumer rights | ADMT Risk Assessment |
| **NIST AI RMF 1.0** | 19 categories, 4 functions | All 19 categories across GOVERN, MAP, MEASURE and MANAGE | AI RMF Profile |

The Quarterly Board Pack, Evidence Pack and Auditor Traceability Matrix span every framework you've
adopted.

## ISO/IEC 42001

**What's in the catalog:** the AI management system requirements in clauses 4–10 (context,
leadership, planning, support, operation, performance evaluation, improvement) and all 38 Annex A
controls (A.2 to A.10), using the standard's own reference numbering.

**What WhiteBoxXAI records:**

- applicability decisions and justifications for every control, which become the **Statement of
  Applicability**
- evidence per control, with administrator approval
- **AIMS policies and guidelines** with an approval step, and **AI objectives**
- assessments you record, and a readiness view by clause with open gaps
- a downloadable certification package (JSON) for your certification body

**Keep in mind:** titles are summaries. You still need the licensed standard for audit-grade wording,
and should check clause citations against your purchased copy before relying on them in a filing.
WhiteBoxXAI doesn't certify you against ISO/IEC 42001; an accredited certification body does.

## EU AI Act

**What's in the catalog:** obligations an operator holds *per AI system* that a governance platform can
evidence, in ten groups: AI literacy and prohibited practices; risk classification; risk management
and data governance; technical documentation and record-keeping; human oversight and robustness;
quality management and the AI value chain; deployer obligations; conformity assessment and
registration; transparency and the right to explanation; and post-market monitoring and incident
reporting.

The catalog follows Regulation (EU) 2024/1689 as amended by the Digital AI Omnibus (Regulation (EU)
2026/1744). Application dates are phased:

| Obligations | Apply from |
| --- | --- |
| AI literacy and prohibited practices (Articles 4 and 5) | 2 February 2025 |
| High-risk requirements (Articles 6–27) for Annex III systems | 2 December 2027 |
| High-risk requirements for Annex I systems | 2 August 2028 |

These dates appear on the compliance calendar as regulatory dates, each with its citation and the date
it was last checked.

**What WhiteBoxXAI records:**

- a **risk classification** per AI system (unacceptable, high, limited or minimal)
- conformity assessments and applicability decisions
- the evidence Annex IV asks for: intended purpose, design, data governance and risk management
  measures, drawn from the system's register entry, audits and monitoring
- explanations (SHAP, LIME, DiCE counterfactuals) that support transparency and the right to
  explanation

**Not covered:** regulator, member-state and notified-body provisions, and the general-purpose AI
model rules (Articles 51–56), which apply to models rather than to each AI system. Confirm current
applicability for your organization before relying on the catalog in a filing.

## GDPR

**What's in the catalog:** the articles a model-governance platform can meaningfully evidence:
principles and lawfulness, transparency and data-subject rights, accountability and design (including
DPIAs and Article 22 automated decision-making), and breach notification. It is not a complete GDPR
compliance program.

**What WhiteBoxXAI records:**

- **data subject requests**, including Article 22 automated-decision requests, newest first
- Article 30 processing records
- a breach register
- consent records as evidence
- explanations of individual automated decisions

## CCPA

**What's in the catalog:** CCPA/CPRA sections and CPPA regulations relevant to **automated
decision-making technology (ADMT)** and the risk assessments the CPPA requires for it, plus consumer
rights. It follows the Civil Code as amended effective 1 January 2026 and the CPPA's approved
regulations.

**What WhiteBoxXAI records:**

- consumer requests
- ADMT applicability decisions and risk-assessment evidence
- fairness audits and explanations for the automated decisions in scope

## NIST AI RMF

**What's in the catalog:** the framework's 19 categories across its four functions: **GOVERN** (6),
**MAP** (5), **MEASURE** (4) and **MANAGE** (4). WhiteBoxXAI works at category level rather than the
72 subcategories, because most subcategories are organizational practices a platform can't evidence
itself.

The NIST AI RMF is voluntary. No category is mandatory, and "in scope" means *you* have declared it in
scope, not that it's a legal requirement.

**What WhiteBoxXAI records:**

- applicability decisions per category, which become the **AI RMF Profile**
- open risks per function, from the [AI Risk Register](../user-guide/risk-register.md)
- the measurement evidence for MEASURE: drift, fairness, explainability and LLM safety results

## One artifact, several frameworks

Most evidence counts towards more than one framework. A bias audit report, for example, maps to
controls in all five. File it once, and **Compliance & Evidence → Controls → Evidence by artifact**
shows every control it supports. See [Compliance & Evidence](../user-guide/compliance-evidence.md#evidence-by-artifact).

## From code and agents

```python
frameworks = client.compliance.list_frameworks()      # key, name, total_controls, adopted, next_audit_date
controls = client.compliance.get_framework_controls("iso_42001")   # gdpr, ccpa, eu_ai_act, nist_ai_rmf
client.compliance.iso42001_score()
client.compliance.iso42001_gaps(severity_filter="high")
```

The MCP server exposes the same data (`whitebox_compliance_*` tools), and the
[compliance-evidence skill](../agent-resources/skill-libraries.md) tells an AI assistant to report
unassessed controls as "not yet assessed" rather than estimating a result.
