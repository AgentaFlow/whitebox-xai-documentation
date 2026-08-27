# Audit & Explanation Reports

You have a deadline — an EU AI Act milestone, an ISO 42001 audit, a customer's vendor
security questionnaire — and you need evidence you can hand to an auditor or put in a board
deck *without* translating it yourself first. This page is that evidence, end to end: what a
report contains, how to generate one, and how to trust what's in it.

!!! note "Why this exists"
    Bias/fairness audits done ad hoc in a data scientist's notebook don't hold up under real
    audit pressure, and a generic GRC tool has nothing at the model level. This is the
    single place WhiteBoxXAI packages model performance, drift, bias, explainability, and
    compliance evidence into an artifact — PDF, CSV, Excel, JSON, HTML, or Markdown — that
    stands on its own.

## Why you can trust what's in a report

Every number in a report traces back to something the platform actually computed, not a
template filled in with placeholder text:

- **Explanations are live SHAP and LIME output**, not canned copy — the same computation
  you could re-run yourself, packaged for someone who isn't going to open a notebook to
  verify it.
- **Governance and risk data is enforced in the database**, not asserted in a policy
  document — a report reflects what the system actually did (who approved what, when a
  control fired), giving you control-to-artifact traceability instead of "we have a policy."
- **Nothing in a compliance or bias-audit report is templated filler.** If a section has no
  underlying data for a model, it says so — it doesn't fabricate a number to fill the page.

If you're validating this for the first time, see the [Explainability
Engine](/user-guide/features/#explainability-engine) and [Bias Detection](/user-guide/features/#bias-detection)
sections of the Features guide for exactly how SHAP, LIME, and fairness metrics are
computed.

## Report categories

| Category | Contents |
| --- | --- |
| **Model performance** | Metrics over time, prediction volume, error analysis, comparison to baseline |
| **Drift analysis** | Data and concept drift, feature distribution changes, time series |
| **Bias audit** | Fairness audit results, per-metric group comparisons, recommendations |
| **Explainability** | SHAP and LIME explanation summaries and feature importance |
| **Compliance** | Regulatory status, audit trail, model documentation |
| **LLM monitoring** | Usage, cost, safety, and quality metrics |
| **Risk register** | [AI Risk Register](/user-guide/risk-register/) entries with owners, scores, and status |
| **Trust score** | [Trust Score](/user-guide/trust-score/) values and component breakdowns |
| **Custom** | A template you define yourself |

**Output formats:** `pdf`, `csv`, `excel`, `json`, `html`, and `markdown`. PDF for sharing
and audit packages, CSV or Excel for further analysis, JSON for feeding another system.

## Generate a report

=== "Dashboard (no code)"

    1. Go to **Governance & Evidence → Evidence & Reports** in the sidebar.
    2. Click **Generate Report**.
    3. Configure the template, the models and date range to include, and the output
       format:

        ```
        Template: Performance Report
        Models: Credit Risk Model
        Date Range: Last 7 Days

        Sections:
          ✓ Executive Summary
          ✓ Prediction Volume
          ✓ Performance Metrics
          ✓ Trend Analysis
          ✓ Alert History

        Detail Level: Standard
        Format: PDF
        ```

    4. Click **Generate Report** and wait — processing takes roughly 30–300 seconds
       depending on data volume, moving through `pending` → `in_progress` → `completed`.
    5. **View Report** to preview it in the browser, **Download** to save it locally, or
       **Share** to send it to someone outside your organization — see [Share a report
       externally](#share-a-report-externally).

    A report includes an executive summary (totals, trend, drift status, recommendations),
    charts (volume over time, accuracy trend, precision/recall, prediction distribution),
    and tables (daily metrics, alert history, top features by importance) — the shape
    varies by template.

=== "API / SDK"

    Report generation lives under `/api/v1/export/*` — note the path, not `/api/v1/reports`,
    which returns `404`.

    ```bash
    # Kick off an export
    curl -X POST https://api.whiteboxxai.com/api/v1/export/exports \
      -H "Authorization: Bearer $WHITEBOXXAI_API_KEY" \
      -H "Content-Type: application/json" \
      -d '{
        "template_id": "...",
        "format": "pdf",
        "model_ids": ["..."],
        "date_from": "2026-07-01",
        "date_to": "2026-07-31"
      }'

    # Check status
    curl https://api.whiteboxxai.com/api/v1/export/exports/{export_id} \
      -H "Authorization: Bearer $WHITEBOXXAI_API_KEY"

    # Download when completed
    curl -L -o report.pdf \
      https://api.whiteboxxai.com/api/v1/export/exports/{export_id}/download \
      -H "Authorization: Bearer $WHITEBOXXAI_API_KEY"
    ```

    Use `POST /api/v1/export/exports/bulk` to queue several exports at once — useful when
    you're assembling a full evidence package across every model in scope for an audit.

    See the [API Reference](/sdk/api-reference/#exports--reports) for the full endpoint
    surface, including templates, configurations, and delivery integrations.

### Evidence Pack: one call, everything in scope

For the common case of "I need everything for this audit right now," skip assembling a
template and call the Evidence Pack endpoint instead:

```bash
curl -X POST https://api.whiteboxxai.com/api/v1/export/evidence-pack \
  -H "Authorization: Bearer $WHITEBOXXAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "format": "pdf",
    "model_ids": ["..."]
  }'
```

It composes Trust Score, AI Risk Register, and Compliance evidence — plus any model-scoped
categories you include — into one branded document, using the same [report
branding](#delivery) as everything else. On the dashboard, it's the **Generate Evidence Pack**
card on the Evidence & Reports page.

## Scheduled reports

Send a report automatically on a recurring basis — the usual pattern for a monthly
compliance package or a weekly performance summary handed to the same stakeholders every
time:

```bash
POST /api/v1/export/scheduled-reports
{
  "name": "Monthly Compliance Package",
  "template_id": "...",
  "format": "pdf",
  "cron_expression": "0 9 1 * *",
  "recipients": ["compliance-team@company.com"]
}
```

Manage schedules with `GET /api/v1/export/scheduled-reports`, `PUT`/`DELETE` on
`/{report_id}`, and trigger one immediately with
`POST /api/v1/export/scheduled-reports/{report_id}/run`.

## Delivery

Reports can be delivered by `download`, `email`, `webhook`, `s3`, `sftp`, or `api`.
Configure third-party delivery integrations under `/api/v1/export/integrations`.

Reports can also carry your own branding — a logo and a primary color, applied to every
generated report. It's self-service: any organization admin sets it under **Settings → Report
Branding**, or through `PATCH /api/v1/organizations/{id}/branding` and
`POST /api/v1/organizations/{id}/branding/logo`.

## Share a report externally

Handing a report to someone outside your organization — an external auditor, a customer's
security team — doesn't mean emailing a PDF or creating them a dashboard account. A share link
authenticates the *recipient*, not just the link:

=== "Dashboard (no code)"

    1. On the Evidence & Reports page, open a completed report and click **Share externally**.
    2. Enter the recipient's email address.
    3. They get an email with a link to a page outside the authenticated dashboard
       (`/share/{share_slug}`). Opening it doesn't show the report — it prompts for a one-time
       passcode, sent to that same email address, before anything is revealed.
    4. Track every share's status (pending, verified, revoked), view count, and expiry from
       the same dialog, and revoke one at any time.

=== "API / SDK"

    ```bash
    # Create a share for a completed export
    curl -X POST https://api.whiteboxxai.com/api/v1/export/exports/{export_id}/share \
      -H "Authorization: Bearer $WHITEBOXXAI_API_KEY" \
      -H "Content-Type: application/json" \
      -d '{"recipient_email": "auditor@theirfirm.com"}'

    # List shares for an export
    curl https://api.whiteboxxai.com/api/v1/export/exports/{export_id}/shares \
      -H "Authorization: Bearer $WHITEBOXXAI_API_KEY"

    # Revoke one
    curl -X POST https://api.whiteboxxai.com/api/v1/export/shares/{share_slug}/revoke \
      -H "Authorization: Bearer $WHITEBOXXAI_API_KEY"
    ```

    The recipient's side of the flow is unauthenticated by design — they don't have a
    WhiteBoxXAI account — and rate-limited:

    ```bash
    POST /api/v1/export/shares/{share_slug}/request-otp   # emails a 6-digit code
    POST /api/v1/export/shares/{share_slug}/verify-otp     # {"otp_code": "123456"}
    GET  /api/v1/export/shares/{share_slug}/download       # Bearer: the token verify-otp returned
    ```

Only a completed export can be shared. A few things worth knowing about the security model:

- The share link itself is a **locator, not a credential** — reaching `/share/{share_slug}`
  reveals nothing. The recipient still has to prove control of their email via a one-time
  passcode before a download token is minted.
- That download token is scoped and short-lived (15 minutes) — it authorizes nothing beyond
  downloading that one report.
- Revoking a share blocks any further OTP request or verification immediately. A download
  token already issued before the revocation remains valid only until its own 15-minute
  expiry.
- Every step — created, code requested, verified, denied, viewed, revoked — is written to the
  audit trail.

## Where this data comes from

Reports pull directly from the features that produce the underlying evidence:

- [Trust Score](/user-guide/trust-score/) — a single 0–100 index over fairness, drift, and
  explainability that a board can read without translation.
- [AI Risk Register](/user-guide/risk-register/) — the structured risk inventory ISO 42001
  and the EU AI Act ask for, with owners and a full audit trail.
- [Governance Review Boards](/user-guide/governance/) — multi-party approval workflows and
  an immutable decision archive.
- [AI Regulations](/account/ai-regulations/) — the regulatory landscape a compliance report
  is measured against.

## Retention and storage

All generated reports are saved and searchable in report history:

- **Free/Starter:** 30 days
- **Professional:** 90 days
- **Enterprise:** Unlimited

## FAQ

**Can I customize what's in a report?** Yes — choose a built-in template (Performance,
Drift, Bias Audit, Compliance, etc.) or define a `custom` one with the exact sections you
need.

**Is a report a point-in-time snapshot or live data?** A generated report is a snapshot as
of the date range you specify — reproducible and stable for an audit trail, but you can
regenerate at any time to reflect the latest data, or set up a schedule to do it
automatically.

**What if a section has no data for a model?** It's marked as such rather than filled with
a placeholder — an empty section is more defensible under audit than a fabricated one.

Still have questions? See the full [FAQ](/help/faq/#reports) or
[Troubleshooting guide](/help/troubleshooting/).
