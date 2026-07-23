# Plans & Limits

WhiteBoxXAI offers several plans so you can start small and scale as your monitoring
needs grow. Your plan determines how many API calls you can make each month and which
features are available to your organization.

## Plan comparison

| Capability | **Demo** | **Free** | **Business** |
| --- | --- | --- | --- |
| **Best for** | Exploring the product | Small projects & evaluation | Teams running models in production |
| **Access** | Read-only | Full access | Full access |
| **Monthly API calls** | Unlimited (shared demo) | 1,000 | Unlimited |
| **Models** | Preloaded samples | Your own | Your own |
| **Explainability (SHAP/LIME)** | View only | ✓ | ✓ |
| **Drift & bias auditing** | View only | ✓ | ✓ |
| **Governance review boards** | View only | Limited | ✓ |
| **Support** | Community | Community | Priority |

!!! note "Plans are evolving"
    The **Business** plan (dedicated workspace and higher limits) is rolling out. If you
    need higher quotas or enterprise features today, [contact us](#need-more) and we'll
    help you find the right fit.

## What each plan includes

### Demo

The demo is a shared, read-only showcase preloaded with realistic sample data — models,
predictions, drift events, bias audits, and reports. It's the fastest way to see what
WhiteBoxXAI can do without connecting any of your own data. Because it's shared and
read-only, you can browse everything but can't create, edit, or delete records.

### Free

A free account gives you full, hands-on access to monitor your own models, with a monthly
allowance of **1,000 API calls**. It's ideal for a single project, a proof of concept, or
learning the platform end to end. When you approach your monthly limit, you'll see it
reflected in your usage and can upgrade at any time.

### Business

The Business plan removes the monthly API-call limit and unlocks the full feature set —
including full governance review-board workflows and priority support — for teams running
models in production.

## Understanding API usage

Most actions you take through the [SDK](../sdk/index.md) or REST API count as API calls —
registering models, logging predictions, requesting explanations, and generating reports.
A few tips to make the most of your allowance:

- **Sample high-volume models.** For models serving thousands of predictions per day, log
  a representative sample (for example, 1–10%) rather than every prediction. You still get
  statistically meaningful monitoring. See
  [Getting Started](getting-started.md#common-questions) for guidance.
- **Batch your logging.** Sending predictions in batches is more efficient than one call
  per prediction. The SDK supports [local buffering and batch flush](../sdk/index.md).
- **Check your usage anytime.** Your current usage and remaining allowance are shown in
  your account settings.

If you exceed your monthly allowance on the Free plan, additional requests are paused until
the next cycle or until you upgrade — your existing data and dashboards remain fully
accessible.

## Need more?

Need higher limits, a dedicated workspace, or enterprise governance and support? Email
**[support@whiteboxxai.com](mailto:support@whiteboxxai.com)** and we'll help you choose the
right plan.
