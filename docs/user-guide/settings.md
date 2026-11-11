---
description: Every WhiteBoxXAI Settings pane — profile and two-factor security, preferences, data and privacy, users and roles, API keys, integrations and notification channels, subscription and usage, trust score weights, brand profiles, writing style, security posture and access reviews, and sample data.
---

# Settings

**Settings** is in the sidebar footer and the avatar menu. Panes are grouped into **You** (your own
account) and **Organisation** (shared settings, mostly for administrators).

## You

### Profile & security

- Name and avatar
- **Change password**
- **Two-factor authentication**: turn it on with an authenticator app and save your backup codes.
  You need it to vote on review requests. See [Two-Factor Authentication](../account/two-factor-authentication.md).
- **Recent activity** on your account

### Preferences

- **Layout**: **New layout** (Enhanced) or **Classic layout**
- **Lens**: your default [lens](index.md#lenses-view-as)
- **Theme**: **Light**, **Dark** or **Match my device**
- **Density**: **Comfortable** or **Compact**

Preferences sync to your account, so they follow you across browsers.

### Data & privacy

- **Marketing emails**: opt in or out
- **Download my data**: a JSON export of the personal data held about you
- **Delete my account**: asks for your password, and an authenticator code if two-factor is on

### Notifications

- **Delivery Channels**: the organization's configured channels (Slack, email, webhook, SMS,
  PagerDuty) and whether each is enabled. Administrators add and test channels under
  **Integrations**.
- **Alert Rules**: a summary of how alert rules work, with **View and configure Alert Rules**, which
  opens [Needs Attention → Alert Rules](needs-attention.md#alert-rules).
- **Defensible Audit Trail & Governance Notices**: how notification activity is recorded.

## Organisation

### Users & roles

**Invite user** sends an email invitation with a role: **Administrator**, **Analyst** or **Viewer**.
The invitee sets their own password from the link; no password is sent by email. Administrators can
change roles and **Revoke** access. You can't change your own role.

See [roles](index.md#roles) for what each role can do.

### API keys

Administrators create, view and revoke API keys here. A key is shown once, when it's created; the
list shows each key's expiry and when it was last used. Non-administrators see who their
administrators are instead.

A key acts with the permissions of the administrator who created it. See [API Keys](../account/api-keys.md).

### Integrations

Health cards for **GitHub**, each notification channel type, the **SDK**, the **LiteLLM gateway** and
the **MCP server**, showing whether each is connected and working. The gateway card covers every
connected LLM gateway, shows the one that most needs attention, and links to the LLM gateway pane.

**Notification channels** are where alerts and notifications are delivered: **Slack**, **email**,
**webhook**, **SMS** and **PagerDuty**. Administrators can add, edit and delete channels, and use
**Send test** to confirm one works before relying on it.

### Subscription & usage

Your **Plan**, its **Status** and when it **Renews**, and usage this month: **N of M Governance
Events used**. On Business Cloud, usage past the allowance shows as billed as overage. Depending on
your plan you'll see **Manage billing**, **Upgrade**, or *"To change this plan, contact billing"*.

See [Plans & Limits](../get-started/plans.md) and [Governance Events](../get-started/governance-events.md).

### Trust score weights

Administrators set how much fairness, drift and explainability each contribute to the
[Trust Score](trust-score.md). Scores are recalculated on a schedule, so new weights show from the
next recalculation.

### Brand profiles

The look of generated reports: colours (with a contrast check), font, cover layout, header and footer
text, confidentiality marking, disclaimer, watermark, page size and logo. Keep several profiles and
set one as the **Default**. Administrators edit; everyone else can view. See
[Reports](reports.md#brand-profiles).

### Writing style

Explains what each lens changes in report wording, and the audience, tone, detail and spelling
options report templates use. See [Reports](reports.md#templates-and-writing-style).

### Security posture

For administrators: an at-a-glance view of your organization's access hygiene.

- **Two-factor authentication**: the share of users with two-factor turned on
- **API keys**: active keys, and stale ones that haven't been used recently
- **Role distribution**: how many users hold each role
- **Access review**: accounts that look stale, the **Last formal review** date, and when the next
  review is due

#### Run an access review

Periodic user access reviews, which auditors routinely ask to see, are run from this pane:

1. **Start Access Review** (or **Continue In-Progress Review**). The review takes a snapshot of
   active user accounts, with each person's role and whether two-factor authentication is enabled.
2. For each account, choose **Keep** or **Revoke**. Revoking requires a note explaining why.
3. Add **Review Notes / Ledger Summary**, for example *"Quarterly access review completed by Security
   Officer"*.
4. **Complete Access Review**. Each decision is recorded as an attestation (retained or revoked), with
   who reviewed it and when.

A completed access review is filed as evidence through **Collect system evidence** in
[Compliance & Evidence](compliance-evidence.md#collect-system-evidence).

### LLM gateway

Administrators connect their organisation's LiteLLM gateway here once. LiteLLM then sends every call
through it to WhiteBoxXAI, which files each under the AI system it belongs to. The pane shows a verdict in
words (receiving, gone quiet, calls being lost), what is stored from each call and what redaction misses,
the configuration to paste into LiteLLM, the traffic no rule has claimed so you can assign it to an AI
system, and the controls to rotate the token, switch acceptance off or disconnect. Non-administrators are
told that administrators manage it.

The pane opens on the **LiteLLM** tab. It also has tabs for OpenTelemetry (OTLP), Portkey AI, Kong AI
Gateway and Azure APIM; each gateway is a separate connection with its own token.

See [LiteLLM gateway](../integrations/litellm.md) and [LLM gateways](../integrations/gateways.md).

### Sample data & tour

- **Load sample data**: fill your account with clearly marked sample models, predictions, drift
  events, audits and reports. Sample-data models don't count against your plan.
- **Reset sample data**: remove them (it asks you to confirm)
- **Product tour**: replay the guided tour
