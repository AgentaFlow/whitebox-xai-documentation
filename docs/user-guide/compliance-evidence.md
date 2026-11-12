---
description: WhiteBoxXAI Compliance & Evidence — framework readiness for ISO/IEC 42001, the EU AI Act, GDPR, CCPA and NIST AI RMF, control applicability decisions, evidence filing and approval, automatic system evidence collection, evidence packs, the compliance calendar and the tamper-evident audit trail.
---

# Compliance & Evidence

**Compliance & Evidence** shows *what an auditor would be handed today, and exactly which controls
would still be missing an artifact*. It works across the five frameworks WhiteBoxXAI covers: ISO/IEC
42001, the EU AI Act, GDPR, CCPA and NIST AI RMF.

WhiteBoxXAI **records evidence; it does not certify compliance.** It shows which controls you've
decided apply, which have approved evidence, and where the gaps are. Whether that evidence satisfies
a regulator or auditor is your organization's call.

Press `g` then `e` to get here. The tabs are **Frameworks**, **Controls**, **Evidence packs**,
**Reports**, **Calendar** and **Audit trail**.

## Frameworks

One card per framework. Each card shows:

- **N of M controls with approved evidence**: always a count, never a percentage
- **Not adopted**, **Assessment not started** or **Start assessment** when there's nothing to count
  yet
- the next **audit date**
- up to three **open gaps**, each with an action to close it, such as **File evidence**, **Run a
  fairness audit**, **Generate a model card**, **Update the risk register**, **Review monitoring**,
  **Review open alerts**, **Review consent records** or **Upload the policy document**

Adopting a framework and setting its audit date are part of the
[readiness setup](overview.md#readiness-setup-wizard).

### Framework detail

**Open the framework** opens a detail panel with framework-specific records:

| Framework | Detail panel |
| --- | --- |
| **ISO/IEC 42001** | Controls by clause (4–10), open gaps, assessments recorded, **Record an assessment**, **Download the certification package** (JSON). **AIMS Policies & Guidelines**, with an **Approve** step for policies pending approval, and **AI Objectives**. |
| **EU AI Act** | Assessments against the EU AI Act, **risk classification** per system, and conformity assessments |
| **GDPR** | Data subject requests (newest first) and the breach register |
| **CCPA** | Consumer requests |
| **NIST AI RMF** | Categories by core function (**GOVERN**, **MAP**, **MEASURE**, **MANAGE**) with open risks per function, open gaps and assessments recorded |

#### ISO/IEC 42001 policies and objectives

The ISO/IEC 42001 panel has registers for **AIMS Policies & Guidelines** (Clause 5) and **AI
Objectives** (Clause 6.2):

1. **Record AIMS Policy**: give the policy a title (for example *Organizational AI Acceptable Use
   Policy*), a **Version** and its **Content**: the policy statements, scope and operational
   requirements. A new policy is **Pending Approval**.
2. **Approve**: an administrator approves a pending policy, and it shows as **Approved**. The
   approval is recorded in the audit trail with the approver's name.
3. **Add AI Objective**: an **Objective Title** (for example *Maintain demographic disparity under
   5%*), a **Description & Criteria** with measurable targets, verification cadence and responsible
   roles, and a **Target Date**.

Approved policies and recorded objectives are evidence for the ISO/IEC 42001 leadership and planning
clauses.

## Controls

The **Controls** tab lists every control for a framework, with **Gaps**, **Undecided** and
**Excluded** filters and a search box (**Search refs or titles**).

| Column | Meaning |
| --- | --- |
| **Control** | The control's reference and title |
| **Applies?** | Your applicability decision |
| **Status** | **Evidenced**, **Gap** or **Excluded** |
| **Evidence** | Evidence filed against the control |
| **Justification** | Why a control was excluded |

### Decide applicability (administrators)

**Decide** (or **Change decision**) records whether a control applies to you. Excluding a control
requires a written justification, which appears in the Statement of Applicability and other
reports. Mandatory controls are marked **Always applies** and can't be excluded. Administrators can
also **Add a custom control** for an internal requirement, and **Deactivate** one later.

### File evidence

**File evidence** uploads an artifact against a control. Artifact types include bias audit report,
impact assessment, model card, risk assessment, data lineage documentation, audit report, policy
document, training record, vendor assessment, monitoring report, incident report and consent
record.

New evidence is **pending** until an administrator approves it: *"An administrator approves it
before it counts."* Only approved evidence counts towards a framework's "N of M".

### Evidence by artifact

One artifact often satisfies controls in several frameworks: a bias audit report maps to controls
in all five. **Evidence by artifact** shows that crosswalk, so you file
once and see every control it supports.

### Collect system evidence

**Collect system evidence** gathers evidence WhiteBoxXAI already holds and files it for review:
bias audits, drift reports, model cards, access reviews and CI/CD lineage. Collected items are filed
as **pending**, so an administrator still decides what counts.

### ServiceNow AI Control Tower

If your change approvals, vendor risk assessments and AI governance records live in ServiceNow AI
Control Tower, you shouldn't have to retype them as evidence. WhiteBoxXAI pulls them in as the
technical evidence behind them.

An administrator turns on the **ServiceNow AI Control Tower** integration first (it appears as a
status card under [Settings → Integrations](settings.md#integrations)). Then **Sync from
ServiceNow AICT**, in **Evidence by artifact**, reads records from your instance. It reads the
`change_request` table by default, up to 50 records at a time (500 at most), and can take a
ServiceNow query to narrow them. The tables it accepts are:

- `sn_ai_governance_asset_governance_details`, `cmdb_ci_appl_ai_application`, `cmdb_ci_function_ai`
  and `alm_ai_dataset_digital_asset`
- `sn_vdr_risk_asmt_assessment` (vendor risk assessments)
- `change_request` (change approvals) and `incident`
- `sn_compliance_policy_exception` and `sn_risk_risk`

What happens to each record:

1. It is stored as an exact JSON snapshot with a **SHA-256 hash**. A record that hasn't changed since
   the last sync is skipped, so syncing twice doesn't file it twice (*"N new evidence artifacts
   created, M unchanged records skipped"*).
2. It is filed as **pending**. Like every other evidence item, it doesn't count towards a
   framework until an administrator approves it, because a ServiceNow ticket existing isn't the
   same as the control being met.

Records can also be pushed to WhiteBoxXAI instead of pulled. Send them to
`POST /api/v1/integrations/servicenow/evidence`, or point a ServiceNow business rule at
`POST /api/v1/integrations/servicenow/webhook`, which does the same thing. Both use your normal
[API key](../account/api-keys.md) or session. See the
[REST API Reference](../sdk/api-reference.md#servicenow-ai-control-tower).

## Evidence packs

**Compose a pack** for **one AI system** or **the whole organization**, as PDF or HTML, and click
**Generate evidence pack**. The pack is saved to [Reports](reports.md) → **History**, where it can
be signed off and shared with an auditor.

Choose the **Reporting period**: **Last 30 days**, **Last 90 days** or **Last 12 months**. The
trust score and the risk register show their state when the pack is generated; the other sections
cover the period you chose.

**What's in this pack** previews the contents before you generate, and changes with your choice:

- **Always included:** a cover with the scope, the period and how many findings sit at each rating;
  for each section, what it shows an auditor and which requirements it is relevant to; and charts
  where there is enough data to draw one, with missing data named as missing.
- **Organization-wide:** every finding with its rating, the trust score across all AI systems, open
  high and critical risks from the register, and the audit trail for the period.
- **Added when you pick one AI system:** performance against the baseline, data and concept drift,
  bias and fairness audits, and explainability coverage.

## Reports

The **Reports** tab is the Report Studio: 12 report types, templates, branding, history with
sign-off, auditor share links and schedules. See [Reports](reports.md).

## Calendar

A month view with **Today** and previous/next navigation, plus an **Agenda**: **Overdue**, **Next
90 days**, **Later**, **Regulatory dates already past** and **Completed**.

| Item | Comes from |
| --- | --- |
| **Audit** | Audit dates you set for each framework |
| **Questionnaire**, **Internal** | Milestones an administrator adds |
| **Regulatory date** | Regulatory milestones, each with its citation and the date it was last checked |
| **Review due** | Scheduled governance reviews |
| **Risk treatment due** | Treatment due dates from the [AI Risk Register](risk-register.md) |

Administrators can add, edit, complete, reopen and delete milestones. Deadlines within your chosen
window also appear in [Needs Attention](needs-attention.md).

## Audit trail

Every recorded action, by whom and when, for a period from **24 hours** to **90 days**:

- **Integrity check**: *N of M events verified*. Each event is chained to the previous one, so an
  altered or deleted event shows up as unverified.
- **Compliance report**: download the period's audit trail as **PDF** or **CSV**
- Panels: **Events in this period**; events **by category**, **severity** and **outcome**; **Most
  active**; **Events over time**; and the **Events list** with filters, search and paging
- **Retention**: how long audit events are kept

## From code

```python
client.compliance.list_frameworks()
client.compliance.get_framework_controls("eu_ai_act")       # a key from list_frameworks()
client.compliance.iso42001_score()
client.compliance.iso42001_gaps(severity_filter="high")
```

Unassessed controls come back marked as not assessed. Report them that way, never as a likely
pass. See [Compliance frameworks](../account/ai-regulations.md) for what WhiteBoxXAI records for each
framework.
