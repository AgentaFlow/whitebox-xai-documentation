# Plans & Limits

WhiteBoxXAI offers four ways to run the platform, from a lifetime free account to a fully
self-hosted deployment. Your plan determines your monthly API allowance, which premium
features you can access, and whether you're on the shared platform or a dedicated workspace.

## Plan comparison

| Capability | **Demo** | **Free** | **Business Cloud** | **Enterprise Edition** |
| --- | --- | --- | --- | --- |
| **Price** | Free | $0 forever | $599/month | Custom (lifetime license) |
| **Best for** | Exploring the product | Evaluation, single projects, hobby use | Teams running models in production | Regulated organizations & governments |
| **Hosting** | Shared platform | Shared platform | Dedicated cloud workspace | Your own cloud or data center |
| **Access** | Read-only | Full access | Full access | Full access |
| **Monthly API calls** | Unmetered | 1,000 | Included allowance + pay-as-you-grow | Unlimited |
| **Models** | Preloaded samples | Your own | Your own | Your own |
| **Monitoring, drift & explainability** | View only | ✓ | ✓ | ✓ |
| **Bias & fairness auditing** | View only | ✓ | ✓ | ✓ |
| **Governance review boards** | View only | ✓ | ✓ | ✓ |
| **Risk Register & Trust Score** | View only | ✓ | ✓ | ✓ |
| **Premium features** | View only | — | ✓ | ✓ |
| **SSO & RBAC** | — | — | Available | ✓ |
| **Air-gapped deployment** | — | — | — | ✓ |
| **Support** | Community | Community | Dedicated support & SLAs | Professional installation & training |

## What each plan includes

### Demo

A shared, read-only showcase preloaded with realistic sample data — models, predictions,
drift events, bias audits, and reports. It's the fastest way to see what the platform does
without connecting any of your own data. Because it's shared and read-only, you can browse
everything, including every premium feature, but you can't create, edit, or delete records.
Usage isn't metered.

### Free

A lifetime free account with full, hands-on access to monitor your own models, with a
monthly allowance of **1,000 API calls**. It's the right fit for a single project, a proof of
concept, or learning the platform end to end. No credit card required.

The free tier doesn't include [premium features](#premium-features), and it has no dedicated
workspace or subdomain. If you exceed the monthly allowance, further API requests are paused
until the next cycle — your existing data and dashboards stay fully accessible.

### Business Cloud

**$599/month.** Your organization gets a **dedicated cloud workspace** at its own subdomain
(`your-company.app.whiteboxxai.com`), with its own database, cache, and storage. Isolation is
physical, not just logical: your data and your compute are separate from every other
customer's.

What's included:

- Dedicated tenant, data isolation, and your own subdomain
- More CPU, GPU, and memory for larger models or higher-volume workloads
- A higher included API allowance, with **pay-as-you-grow billing** past it
- All [premium features](#premium-features), including AI-driven architecture review
- GRC and audit logging
- SSO and RBAC available
- Priority access to new features and beta programs
- Dedicated support and SLAs

#### Signing up

Business Cloud is self-serve. After checkout, your workspace is provisioned automatically —
this takes roughly **10 to 30 minutes**, since a dedicated environment is being built for
you. You'll get a verification email once it's ready, and you'll sign in at your own
subdomain rather than the shared platform.

!!! note "Rolling out"
    Business Cloud is being rolled out in stages. If the pricing page shows **Coming Soon**
    instead of a checkout button, self-serve signup isn't live in your region yet — [contact
    us](#need-help-choosing) and we'll get you set up.

#### How overage billing works

Business Cloud includes a monthly API-call allowance. Unlike the Free plan, going past it
**doesn't block your requests** — they're allowed through and the overage is billed as
metered usage on your next invoice. This is deliberate: a production monitoring pipeline
shouldn't stop logging predictions because it had a busy month.

Your current usage and remaining included allowance are shown in your account settings.

### Enterprise Edition

Built and hosted entirely on **your own cloud or data center**, for regulated organizations
and governments. Priced as a lifetime license with optional yearly maintenance and support.

- Deployed on your infrastructure — full control over your data security
- Air-gapped deployment for high-trust environments
- Maximum CPU, GPU, and memory for your largest models
- ISO 42001, NIST AI RMF, GDPR, EU AI Act, and CCPA governance features
- SSO, RBAC, and audit trails
- All premium features, plus additional features available for regulated industries
- Professional installation, support, and hands-on training

[Contact us](#need-help-choosing) to discuss an Enterprise Edition deployment.

## Premium features

Three capabilities are gated by plan. Business Cloud and Enterprise Edition include all of
them; the Demo lets you view them; the Free plan doesn't include them.

| Feature | What it does |
| --- | --- |
| **Architecture review** | AI-driven review of your model architecture and its governance implications. |
| **Model diff** | Structured comparison between model versions, including changes tied to pull requests. |
| **Local model inference** | Live attention-weight extraction from a self-hosted model, rather than the heuristic path available on every tier. |

!!! note
    Local model inference is not yet wired to a live endpoint. The tier grants access to the
    capability; availability will be announced when it ships.

## Understanding API usage

Most actions through the [SDK](../sdk/index.md) or [REST API](../sdk/api-reference.md) count
as API calls — registering models, logging predictions, requesting explanations, and
generating exports. A few ways to make the most of your allowance:

- **Sample high-volume models.** For a model serving thousands of predictions per day, log a
  representative sample (1–10%) rather than every prediction. You still get statistically
  meaningful monitoring. See [Getting Started](getting-started.md#common-questions).
- **Batch your logging.** Sending predictions in batches costs far fewer calls than one per
  prediction. The SDK supports [local buffering and batch flush](../sdk/index.md).
- **Use offline buffering.** The SDK can queue operations locally and sync them later — see
  [Offline Mode](../sdk/offline-mode.md).
- **Check your usage anytime.** Current usage and remaining allowance are in your account
  settings.

## Need help choosing?

Email **[support@whiteboxxai.com](mailto:support@whiteboxxai.com)** and we'll help you find
the right plan — including higher limits, a dedicated workspace, or a self-hosted
deployment.
