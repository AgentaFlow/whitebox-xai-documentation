---
description: The WhiteBoxXAI Overview — current posture tiles, framework readiness, what changed, trends, value and impact, the five-step readiness setup wizard, and presentation mode for board meetings.
---

# Overview

**Overview** answers *where do our AI systems stand, and what needs a decision first?* It opens on
**Now**, with two more tabs: **Trends** and **Value & impact**. Its two header buttons are **Set up
readiness** and **Present**.

## Now

### Readiness setup checklist

Until setup is complete, a card shows **Your readiness setup: N of 5 steps done**, with **Start
setup** or **Continue setup**. Each step shows **Done**, **To do** or **Could not be checked**:

1. Adopt a compliance framework
2. Set your next audit date
3. Review your control defaults
4. Connect your first model
5. Invite a reviewer

Steps are checked against what's actually recorded. "Connect your first model" is done when real
data arrives, not when a box is ticked. Once all five are done the card reads **Readiness setup
complete**, with **Review setup**. If analysts have proposed changes, the card says how many are
waiting for an administrator.

### Attention band

A band across the top says how many items need attention, broken down by severity, with a countdown
to the nearest deadline. **Review all** opens [Needs Attention](needs-attention.md).

### Posture tiles

| Tile | What it counts |
| --- | --- |
| **Models monitored** | AI systems in the register that are being monitored |
| **Average trust score** | The mean [Trust Score](trust-score.md) (out of 100) across systems that have one |
| **Needs attention** | Open items in the Needs Attention queue |
| **Controls evidenced** | Controls with approved evidence, shown as "N of M" for adopted frameworks |

Every tile has a **How was this computed?** popover naming the records it counted.

### Framework readiness

One row per framework: **N of M controls evidenced**, or **Not adopted** / **Assessment not
started**. Readiness is always shown as a count of evidenced controls, never as a percentage that
could be read as "compliant". **Open Compliance & Evidence** goes to the detail.

### Needs a look first

The systems most in need of attention, with a link to the [Model Register](model-register.md).

### What changed

Changes since a given date: average trust score, findings, decisions, evidence, systems added,
exports completed and audit events recorded.

## Trends

Pick a range (**Last 30**, **90** or **180 days**, or **Last year**) to see:

- Average trust score
- Controls evidenced, by framework
- Findings opened and closed
- Drift incidents

Each chart has a table view with the same numbers, for screen readers and for copying into a report.

## Value & impact

Written for executives and boards: **ROI and value**, **Drift impact** and **Agentic governance**.
The Executive lens opens the Overview on this tab.

**Business Value & Return on Investment** shows **Financial & Usage Telemetry** (**Combined AI
Spend**, **LLM Token Spend**, **Agentic Workflow Spend** and **Predictions Logged**) next to the
return figures. The return figures, such as net ROI and human hours saved, are computed only from
assumptions your organization has set. Where nothing is set, the tile says it isn't instrumented
and why, rather than showing an estimate. An invented ROI number is the kind of figure a board
asks you to defend.

The assumptions are the organization defaults and per-system profiles behind
`GET` and `PUT /api/v1/executive-dashboard/value-config`:

| Level | What you set |
| --- | --- |
| Organization | Hourly labor rate (USD), and minutes saved per prediction, per LLM request and per agent workflow run |
| Each AI system | Governed business goal, monthly infrastructure cost, target monthly volume, minutes saved per unit, monthly revenue at risk, maximum churn risk, latency SLO and monthly operating cost |

The editing dialog, **Configure Value & KPI Assumptions**, is on the Classic layout's Executive
Dashboard (**ROI & Value** tab). The new layout doesn't have the dialog yet, so use the API or
switch layout under **Settings → Preferences** to change them.

## Readiness setup wizard

**Set up readiness** (or the first-run prompt **Set up your compliance readiness**) opens a
five-step wizard: **Frameworks → Audit dates → Applicability → Connect the SDK → Invite a reviewer**.

What you can do depends on your role:

| Role | In the wizard |
| --- | --- |
| **Administrator** | Adopts frameworks, sets audit dates, marks control defaults reviewed (**Mark defaults reviewed**), confirms or rejects analysts' proposals, and invites a reviewer directly |
| **Analyst** | Proposes framework adoptions and dates. *"You propose; an administrator confirms."* |
| **Viewer** | Sees progress only |

Inviting a reviewer sends an email invitation with the role you choose (**Analyst**, **Viewer** or
**Administrator**). The invitee sets their own password from the link.

The first-run prompt appears once per browser for administrators and analysts while setup is
unfinished.

## Presentation mode

**Present** turns the Overview into full-screen slides for a board or committee meeting:

1. **Where we stand**
2. **Compliance readiness**
3. **What changed**
4. **Decisions**
5. **Trends & trajectory**: trust score, findings, drift incidents and evidence coverage, week by
   week

Use the arrow keys, Page Up / Page Down or Space to move between slides, **Home** and **End** to
jump to the first or last, or **1** to **5** to go straight to a slide. **Fullscreen** (where your
browser allows it) fills the screen, **Print** produces one slide per page, and **Exit** or `Esc`
returns to the dashboard. The slides use the same live numbers as the
Overview, so what you present matches what an auditor would see.

For a document to circulate rather than present, generate a **Quarterly Board Pack** from
[Reports](reports.md), or use **⌘K → Generate board pack**.
