---
description: How the WhiteBoxXAI Enhanced Dashboard is organized — sidebar sections, lenses, roles, the command palette, keyboard shortcuts, saved views, and switching between the Enhanced and Classic layouts.
---

# The Enhanced Dashboard

The Enhanced Dashboard is organized around the questions a risk or compliance team gets asked:
*Where do our AI systems stand? What needs a decision? What would we hand an auditor today?* Each
section answers one of them, and every number it shows comes with a **How was this computed?** note
that says exactly what data produced it.

It's the default layout for every account. This page explains how it's laid out; the pages under
**Enhanced Dashboard** in the navigation cover each section in depth.

## The sidebar

| Group | Section | Use it to | Guide |
| --- | --- | --- | --- |
| **Focus** | **Overview** | See where your AI systems stand, run the readiness setup, present to a board | [Overview](overview.md) |
| | **Needs Attention** | Work through alerts, risks, reviews and deadlines, worst first | [Needs Attention](needs-attention.md) |
| **AI Systems** | **Model Register** | Browse every governed AI system and open its dossier | [Model Register](model-register.md) |
| | **Assurance** | Explainability, fairness and drift for a system, each with a verdict | [Assurance](assurance.md) |
| | **Observability** | Runtime performance, LLM & RAG activity, agents and prompt evaluations | [Observability](observability.md), [Evaluations](evaluations.md) |
| **Evidence & Governance** | **Compliance & Evidence** | Frameworks, controls, evidence, reports, the calendar and the audit trail | [Compliance & Evidence](compliance-evidence.md), [Reports](reports.md) |
| | **Reviews & Decisions** | Review requests, board votes, the decision archive and RACI | [Reviews & Decisions](governance.md) |
| | **Risk Register** | AI risks scored by likelihood and impact | [AI Risk Register](risk-register.md) |

Badges on the sidebar count what's waiting: the **Needs Attention** queue total, **your** pending
reviews, and open risks. The footer links to these docs, **Support** and **Settings**.

The header shows your organization, when the data was last refreshed, **Search ⌘K**, a theme
toggle, and a bell that opens Needs Attention with the number of active alerts. When sample data is
loaded, a **Demo data — illustrative only** label appears so nobody mistakes it for your own.

## Lenses: "View as"

Different people read the same evidence differently. The **View as** menu switches the lens:

- **Compliance Officer**
- **Data Scientist**
- **Executive**
- **Auditor**
- **Admin** (administrators only)

A lens *"changes which columns and wording lead. It never changes what you can do."* For example,
the Data Scientist lens leads the Model Register with drift and fairness-gap columns, the Compliance
Officer lens with risk tier, evidence and attestation, and the Executive lens opens the Overview on
**Value & impact**. Your role, not your lens, decides what you're allowed to do.

The lens also shapes report wording: see **Settings → Writing style**.

## Roles

| Role | Can |
| --- | --- |
| **Administrator** | Everything, including users, API keys, framework adoption, control decisions, evidence approval, board setup and scales |
| **Analyst** | Work with systems and evidence, mark verdicts reviewed, file evidence, and *propose* readiness changes for an administrator to confirm |
| **Viewer** | Read everything they have access to |

Some actions depend on more than your role. You can't vote on a review request you raised, and
voting needs two-factor authentication and an authenticator code. See
[Reviews & Decisions](governance.md).

## Search and commands (⌘K)

Press **⌘K** (macOS) or **Ctrl+K** (Windows, Linux) to open **Quick Actions**. Type to search:

- **Systems, controls, generated reports and review requests**, by name
- **Commands**, such as **Generate board pack**, **Open *framework* gaps** for each framework, or
  **Take the product tour**
- **Create** entries: register a model, create an alert rule, generate a report
- **Navigation** and **Settings & Help**

## Keyboard shortcuts

| Keys | Action |
| --- | --- |
| `g` then `o` | Go to Overview |
| `g` then `a` | Go to Needs Attention |
| `g` then `r` | Go to Model Register |
| `g` then `e` | Go to Compliance & Evidence |
| `g` then `v` | Go to Reviews & Decisions |
| `⌘K` / `Ctrl+K` | Search and run commands |
| `?` | Show the keyboard shortcuts list |

## Saved views

On the **Model Register** and the **Needs Attention** queue, set the filters and columns you want
and choose **Saved views → Save current view...**. Saved views are:

- private to you
- kept per lens, so your Auditor views and Data Scientist views stay separate
- synced across your browsers
- limited to 25 per account across all screens

## "How was this computed?"

Every tile, verdict and chart that summarises your data has a **How was this computed?** control.
It says which records were counted, over what period, and with what method. When there isn't enough
data, the dashboard says **Not instrumented**, **Not available** or **Assessment not started**
instead of showing a number. WhiteBoxXAI never fills a gap with an estimate.

## Product tour

The product tour walks through Overview, Needs Attention, Model Register, Assurance,
Observability, Compliance & Evidence, Reviews & Decisions, and search and shortcuts. Start it from
**⌘K → Take the product tour** or **Settings → Sample data & tour**.

## Enhanced and Classic layouts

The **Classic** layout, the dashboard before the Enhanced redesign, remains available for now.

- **Switch to Classic**: avatar menu → **Switch to Classic layout**, or **Settings → Preferences →
  Layout → Classic layout**.
- **Switch to Enhanced**: the **Try the new layout** button in the Classic header (shown on wider
  screens), or **Settings → Preferences → Layout → New layout**.

Your choice syncs to your account. Links to Classic pages open the matching Enhanced section or tab
when you're on the Enhanced layout, so bookmarks keep working.

## Preferences

**Settings → Preferences** sets your layout, default lens, theme (**Light**, **Dark** or **Match my
device**) and density (**Comfortable** or **Compact**). See [Settings](settings.md) for every pane.
