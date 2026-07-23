# Two-Factor Authentication (2FA)

Two-factor authentication adds a second layer of security to your WhiteBoxXAI account. In
addition to your password, you'll enter a short code from an authenticator app when you log
in — so your account stays protected even if your password is ever compromised.

WhiteBoxXAI uses **TOTP** (time-based one-time passwords), the same standard supported by
apps like Google Authenticator, Authy, Microsoft Authenticator, and 1Password.

!!! tip "Recommended for every account"
    We strongly recommend enabling 2FA — and requiring it for any administrator accounts in
    your organization.

## Enable 2FA

1. Log in and go to **Profile → Security**.
2. Click **Enable Two-Factor Authentication**.
3. **Scan the QR code** with your authenticator app. (Prefer not to scan? Enter the setup
   key shown beneath the code manually.)
4. Enter the current **6-digit code** from your app and click **Verify and Enable**.
5. **Save your backup codes** — download or print them and store them somewhere safe (a
   password manager is ideal). Each code can be used once.

Your account is now protected with 2FA.

## Log in with 2FA

1. Enter your email and password as usual.
2. When prompted, open your authenticator app and enter the current 6-digit code.
3. Click **Verify**.

Codes rotate every 30 seconds, so always enter the one currently shown.

## Use a backup code

If you don't have your authenticator app handy — for example, you've replaced your phone —
you can log in with a backup code:

1. Enter your email and password.
2. On the 2FA prompt, choose **Use a backup code**.
3. Enter one of your saved backup codes.
4. Once logged in, go to **Profile → Security** and set up your authenticator again.

You can regenerate a fresh set of backup codes at any time from **Profile → Security**;
generating new codes invalidates the old set.

## Disable 2FA

Go to **Profile → Security → Disable Two-Factor Authentication** and confirm with your
password. We only recommend this if you're switching to a new device — re-enable 2FA as
soon as you can.

## API keys and 2FA

2FA protects interactive logins to the dashboard. It does **not** affect API keys, so your
[SDK](../sdk/index.md) and API integrations keep working without any changes. Continue to
keep API keys secret and rotate them if you suspect they've been exposed.

## Lost access to your account?

If you've lost both your authenticator app and your backup codes, contact
**[support@whiteboxxai.com](mailto:support@whiteboxxai.com)**. Account recovery requires
identity verification and may take 24–48 hours, so keep your backup codes safe.
