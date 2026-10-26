---
description: The WhiteBoxXAI Needs Attention queue — alerts, risks, reviews and deadlines ranked worst first, with filters, saved views and row actions; plus All Alerts and Alert Rules.
---

# Needs Attention

**Needs Attention** gathers everything that needs a person into one queue, worst first. It has three
tabs: **Needs Attention**, **All Alerts** and **Alert Rules**. Press `g` then `a` to get here, or
click the bell in the header.

## The queue

The queue combines four sources:

| Source | Examples |
| --- | --- |
| **Alerts** | Active alerts on a model's performance, drift or safety |
| **Risks** | Open entries in the [AI Risk Register](risk-register.md) |
| **Reviews** | Review requests waiting for a decision |
| **Deadlines** | Audit dates, regulatory dates and review or risk-treatment due dates from the compliance calendar |

### Filter it

- **Source** and **Severity**, each with a count
- **Deadlines due within**: 7 to 90 days

Save a filter combination you use often with **Saved views**. See
[saved views](index.md#saved-views).

### Act on a row

Depending on what the row is: **Acknowledge**, **Assign to me**, **Open review**, **Open risk**,
**Open dossier** or **Open compliance**. Acknowledging and resolving alerts are recorded in the
audit trail under your name.

## All Alerts

Every alert for your organization, with **Total Alerts**, **Active**, **Acknowledged** and
**Resolved** counts, a search box, and status and severity filters. **Acknowledge** and
**Resolve** are on each alert.

## Alert Rules

Alert rules describe the conditions you want to be warned about: a metric, an operator (`gt`, `gte`,
`lt`, `lte`, `eq`, `ne`) and a threshold, with a severity. The tab lists each rule's **Name**,
**Type**, **Severity**, **Status**, **Conditions**, **Times Triggered** and **Created** date.
**Create Alert Rule** opens the rule form.

!!! warning "Alert rules are not evaluated automatically yet"
    Alert rules are stored and can be tested against metric values you supply, using
    `client.alerts.evaluate_rule(rule_id, metric_values={...})` or the
    `whitebox_alerts_evaluate_rule` MCP tool. WhiteBoxXAI does not yet evaluate rules
    automatically against incoming data, so a rule on its own won't raise alerts. Until it does,
    track drift through [drift reports](assurance.md#drift) and the queue's other sources, and keep
    your own thresholds under review.

Choose thresholds yourself. WhiteBoxXAI doesn't present a default threshold as a regulatory
requirement.

## Notifications

Delivery channels (Slack, email, webhook, SMS and PagerDuty) are set up by an administrator in
**Settings → Integrations**, where each channel has **Send test**. See [Settings](settings.md).
