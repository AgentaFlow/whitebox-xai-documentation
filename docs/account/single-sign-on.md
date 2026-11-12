---
description: Connect Microsoft Entra ID so your team signs in to WhiteBoxXAI with their work accounts. Covers who can set it up, what Require SSO blocks, break-glass administrators, just-in-time accounts and role mapping.
---

# Single Sign-On (Microsoft Entra ID)

When an auditor asks who can reach your AI governance evidence, "everyone with a password we
issued" is a hard answer to defend. Single sign-on moves that control to your identity
provider: people sign in with their Microsoft work account, and leaving the company or the
directory removes their access.

WhiteBoxXAI supports **Microsoft Entra ID** (OpenID Connect). It is available on **Business
Cloud** and above; it isn't part of the Free plan. See [Plans & Limits](../get-started/plans.md).

## Before you start

You need three things in place:

- **The WhiteBoxXAI `admin` role.** Only administrators can see or change the connection.
- **[Two-factor authentication](two-factor-authentication.md) turned on for your own account.**
  Starting the connection without it returns *"Enable two-factor authentication before
  connecting single sign-on."*
- **A Microsoft account that is a Global Administrator, Application Administrator or Cloud
  Application Administrator in your Entra tenant.** Connecting grants consent for the whole
  tenant, and WhiteBoxXAI checks the administrator role in the sign-in token. A regular
  member account is refused.

## Connect Entra ID

1. Go to **Settings → Single sign-on**.
2. Click **Connect Microsoft Entra ID**.
3. Sign in with the Microsoft administrator account and approve the consent request.

WhiteBoxXAI takes your tenant from the signed Microsoft token, not from a value you type.

## What people see

Once connected, the sign-in page offers **Sign in with your organisation**. It starts the
Microsoft sign-in and returns the person to WhiteBoxXAI.

## Accounts and roles

| Setting | What it does |
|---|---|
| **Create accounts on first sign-in** | On (the default): a person with a valid Microsoft sign-in gets a WhiteBoxXAI account the first time they arrive. Off: only people an administrator has already added can sign in. |
| **Role when Microsoft sends none** | The role given when the sign-in carries no app role: `viewer` (the default), `analyst` or `admin`. |
| **Role mapping** | Maps the Microsoft **app role** on the sign-in to a WhiteBoxXAI role. The defaults are `WhiteBox.Admin` → `admin`, `WhiteBox.Analyst` → `analyst` and `WhiteBox.Viewer` → `viewer`. You can change the mapping. |

A member's role follows their Microsoft app role **each time they sign in**. Change it in
Entra and it takes effect at their next sign-in.

An existing WhiteBoxXAI account is linked to its Microsoft identity only when Microsoft
reports the email domain as verified. This stops someone with an unverified address from
taking over an account that already exists.

## Require SSO

By default, SSO is an option alongside passwords. **Require SSO** makes it the only way in.

Turning it on refuses, for every member except the break-glass administrators below:

- password sign-in
- the password-and-two-factor flow
- the email-verification sign-in link
- social sign-in
- password reset

Sessions that are already open end within 30 minutes. **API keys are not affected**, so
[SDK](../sdk/index.md) and [MCP](../integrations/mcp.md) integrations keep working.

WhiteBoxXAI won't let you turn it on until:

- the connection is active,
- at least one **break-glass administrator** is chosen, and
- you have signed in with SSO yourself at least once. Otherwise you could lock yourself out
  with a configuration that doesn't work.

### Break-glass administrators

If Entra is down or misconfigured, nobody can sign in with SSO. Break-glass administrators are
the exception: they keep **password and two-factor** sign-in while Require SSO is on. Only
organization administrators with two-factor authentication enabled can be chosen. Pick more
than one if you can, and test their password sign-in before you need it.

## For integrators

The connection is managed through these endpoints, which require an administrator on a plan
that includes SSO. See the [REST API Reference](../sdk/api-reference.md#single-sign-on).

| Method | Path | Notes |
|---|---|---|
| `GET` | `/api/v1/sso/connection` | Read the connection |
| `POST` | `/api/v1/sso/connection/entra/start` | Begin the Entra connect flow |
| `PATCH` | `/api/v1/sso/connection` | Change JIT, default role, role mapping, Require SSO |
| `DELETE` | `/api/v1/sso/connection` | Remove the connection |
| `GET` | `/api/v1/sso/entra/login` | Start a sign-in (public) |
| `GET` | `/api/v1/sso/entra/callback` | Microsoft redirect target (public) |
