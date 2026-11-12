---
description: WhiteBoxXAI plans — Free, Business Cloud and Enterprise Edition — priced in Governed AI Systems and Governance Events, with the same governance features on every tier.
---

# Plans & Limits

Every plan gets the same governance features: explainability, drift, bias audits, LLM and agent
monitoring, the risk register, review boards, and evidence for all five frameworks (ISO/IEC 42001,
GDPR, CCPA, the EU AI Act and NIST AI RMF). What changes between plans is **how many AI systems you
govern**, **how much activity is recorded**, **where the platform runs**, and **what support you
get**.

Plans are sold in two units:

- **Governed AI System**: a model, LLM app or agent system you register, counted by name.
- **Governance Event**: one unit of recorded AI activity, such as a logged prediction, an LLM
  interaction or a bias audit. Configuration, reports, alerts, dashboard use and every read are
  free. See [Governance Events](governance-events.md) for the exact counting rules.

## Plan comparison

| | **Free** | **Business Cloud** | **Enterprise Edition** |
| --- | --- | --- | --- |
| **Price** | $0, no credit card | $599/month, or $5,990/year (2 months free) | Lifetime license plus yearly maintenance & support |
| **Who it's for** | Developers, ML engineers and risk leads evaluating WhiteBoxXAI | Teams governing AI systems in production | Organizations that need full data sovereignty |
| **Governed AI Systems** | 1 | Up to 10 (contact us for more) | Unlimited |
| **Governance Events included** | 2,500 / month | 100,000 / month | Negotiated |
| **Beyond the allowance** | Recording pauses until next month | $5 per 10,000 events, billed monthly | Volume pricing |
| **Where it runs** | Shared WhiteBoxXAI cloud | Your own dedicated tenant at `<your-org>.app.whiteboxxai.com` | Your VPC, cloud, data center or air-gapped network |
| **Compliance frameworks** | All five | All five | All five |
| **Explainability, drift & bias** | ✓ | ✓ | ✓ |
| **LLM, RAG, agent & prompt governance** | ✓ | ✓ | ✓ |
| **MCP server & agent skills** | ✓ | ✓ | ✓ |
| **Model-diff architecture reviews** | — | ✓ | ✓ |
| **Access & audit** | Role-based access, audit log, Google or Microsoft sign-in | Same, plus Microsoft Entra ID single sign-on | Runs inside your own identity, key and network boundary |
| **Support** | Community & documentation | Standard support, business-day response | 24/7 support & custom SLA |

## Free

A lifetime free account for governing your first AI system, with full hands-on access: the REST
API, Python SDK and MCP server, explainability, drift and bias audits, LLM and RAG monitoring,
safety scans, prompt evaluations, the risk register, review boards and evidence for all five
frameworks.

- **1 Governed AI System.** New versions of that system don't count against the limit.
- **2,500 Governance Events per month**, as a hard cap. When the month's allowance is used,
  requests that would record new activity are refused with a `429 QuotaExceeded` response until the
  next month. Everything already recorded stays visible, and the dashboard, reports and reads keep
  working.

Sign up at [whiteboxxai.com](https://whiteboxxai.com/register). Only your name, email and a
password are needed, or you can sign in with Google or Microsoft.

## Business Cloud

**$599/month, or $5,990/year.** Your organization gets its **own dedicated WhiteBoxXAI tenant**,
hosted in Microsoft Azure, at its own subdomain (`<your-org>.app.whiteboxxai.com`). Your data is
stored and processed separately from every other customer's.

- **Up to 10 Governed AI Systems.** Contact us if you need more.
- **100,000 Governance Events per month.** Past that, recording **is not blocked**: extra events
  are billed at $5 per 10,000, monthly. A production monitoring pipeline shouldn't stop logging
  because it had a busy month.
- **Model-diff architecture reviews** that compare model versions and the code changes between
  them. See [GitHub Diff Reviews](../integrations/github-diff.md).
- **Single sign-on with Microsoft Entra ID**, with an option to require it for everyone except
  break-glass administrators. See [Single Sign-On](../account/single-sign-on.md).
- **Standard support**, with a response within one business day.

Business Cloud is available now. [Contact us](https://marketing.whiteboxxai.com/getting-started/demo)
and we'll set up your dedicated tenant with you.

## Enterprise Edition

WhiteBoxXAI deployed on **your own infrastructure**: your VPC, cloud account, data center or an
air-gapped network. Your data, keys and network controls never leave your boundary. It's priced as
a lifetime license, plus yearly maintenance and support where you need it.

- Unlimited Governed AI Systems, with negotiated event volumes
- Everything in Business Cloud
- Sized for your workloads
- Professional installation, 24/7 support and hands-on training

[Request a demo](https://marketing.whiteboxxai.com/getting-started/demo) to discuss an Enterprise
Edition deployment.

## Demo accounts

A read-only demo account, preloaded with sample models, predictions, drift events, bias audits and
reports, is available on request from the [pricing page](https://whiteboxxai.com/pricing). Nothing
can be created or changed in a demo account, and its usage isn't metered.

You can also load the same kind of sample data into your own account from **Settings → Sample data
& tour**. Sample-data models don't count as Governed AI Systems.

## How long data is kept

Raw telemetry (logged predictions, metrics and LLM call logs) is kept for a set period and then
purged. The period depends on the plan:

| | **Free** | **Business Cloud** |
| --- | --- | --- |
| **Raw telemetry** | 90 days, fixed | 365 days by default; an administrator can set it anywhere from 1 to 7 years (365 to 2,555 days), per AI system or for the whole organization |

Demo accounts aren't purged. Calls older than 30 days are compressed in storage, which doesn't
remove them.

The records an auditor relies on aren't part of that purge. Audit-trail events, evidence packs,
drift reports and bias audits are kept. When a purge removes the predictions behind an older drift
report or bias audit, that report stays and is marked: *"Source predictions aged out under
retention policy; metrics are locked and cannot be recomputed."* Each purge is itself written to the
[audit trail](../user-guide/compliance-evidence.md#audit-trail).

## Check your usage

**Settings → Subscription & usage** shows your plan, its status and renewal date, and how many
Governance Events you've used this month out of your allowance. On Business Cloud, usage past the
allowance is shown as **billed as overage** rather than refused.

## Need help choosing?

Email **[support@whiteboxxai.com](mailto:support@whiteboxxai.com)** and we'll help you find the
right plan, including more systems, a dedicated tenant, or a self-hosted deployment.
