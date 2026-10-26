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

Personal notification preferences, such as quiet hours and digests, aren't available yet. Where
alerts and notifications are delivered is set for the whole organization under **Integrations**.

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

Health cards for **GitHub**, each notification channel type, the **SDK** and the **MCP server**,
showing whether each is connected and working.

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

Access reviews are recorded through the API; the pane shows their status. A completed access review
can be filed as evidence through **Collect system evidence** in
[Compliance & Evidence](compliance-evidence.md#collect-system-evidence).

### Sample data & tour

- **Load sample data**: fill your account with clearly marked sample models, predictions, drift
  events, audits and reports. Sample-data models don't count against your plan.
- **Reset sample data**: remove them (it asks you to confirm)
- **Product tour**: replay the guided tour
