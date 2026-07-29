# Governance Review Boards

Governance Review Boards bring executive oversight to your AI models through multi-party
approval workflows, AI-assisted review templates, a searchable decision archive, and
automated periodic reviews. Use them to make sure the right people sign off before a model
goes to production — and to keep an auditable record of every decision.

You can manage everything from the **Governance & Evidence** area of the dashboard, or
automate it through the REST API. API examples below use the base URL
`https://api.whiteboxxai.com` and require an [API key](../account/api-keys.md)
(`Authorization: Bearer YOUR_API_KEY`).

## Core concepts

**Review board** — a group of people who vote on requests, with configurable voting rules
(approval threshold, quorum, and optional weighted or unanimous voting).

**Review request** — something that needs approval (for example, deploying a model),
submitted to a board for a vote.

**Decision** — a member's vote on a request: approve, deny, abstain, or request changes.

**Archive** — every finalized decision is stored with an executive summary and is
full-text searchable for later audits.

### Board types

| Type | Use for |
| --- | --- |
| `technical` | Technical architecture reviews |
| `business` | Business impact assessments |
| `compliance` | Regulatory compliance reviews |
| `executive` | C-level strategic decisions |

### Request types

`model_deployment`, `architecture_change`, `policy_update`, `risk_assessment`,
`compliance_review`.

### Priority levels

`low`, `medium`, `high`, `critical`.

## Create a board

In the dashboard, go to **Governance → Review Boards → New Board**, or use the API:

```bash
curl -X POST https://api.whiteboxxai.com/api/v1/governance/review-boards \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "AI Ethics Committee",
    "description": "Reviews high-risk AI deployments",
    "board_type": "compliance",
    "approval_threshold": 66,
    "quorum_required": 3
  }'
```

### Voting rules

- **`approval_threshold`** — percentage of votes needed to approve (e.g. `66` for a
  two-thirds majority).
- **`requires_unanimous`** — if `true`, every vote must be *approve*.
- **`quorum_required`** — minimum number of votes before a request can be finalized.
- **`voting_power`** — per-member weight, for weighted voting.

## Add board members

```bash
POST /api/v1/governance/review-boards/{board_id}/members
{
  "user_id": "...",
  "role": "member",
  "voting_power": 1.0
}
```

Roles determine what a member can do:

- **Observers** — read-only, cannot vote.
- **Members** — can vote and comment.
- **Chairs** — full board management, including finalizing decisions.

## Governance guarantees

Three rules are enforced at the database layer, not just in the application. That distinction
matters in an audit: "show me the schema" is a stronger answer than "trust our code," and it
means the rules hold even for a caller that bypasses the application entirely.

### Separation of duties

**A request's submitter cannot vote on their own request.** This is enforced by a database
trigger on the decisions table as well as in the service layer, so it can't be circumvented
by calling the API directly rather than using the dashboard. A member can also only cast one
vote per request.

This is the property that makes a review board *governance* rather than an approval button —
and it's why a "zero self-approvals" statement about your organization is provable rather
than asserted.

### Vote changes are recorded, never overwritten

A member may change their vote while a request is still open. When they do, the previous vote
is preserved in an append-only history table rather than being replaced. Nothing about a
vote's history is silently lost.

Once a request is finalized, votes are closed.

### The decision archive is immutable

Finalized decisions cannot be edited or deleted — `UPDATE` and `DELETE` are blocked at the
database level for both the archive and the vote history.

If a decision was recorded in error, you don't mutate history. You record a **correction**: a
new archive entry that supersedes the original, carrying a reference to it (`supersedes_id`)
and a `correction_reason`. Both entries remain, and the chain between them is explicit — which
is exactly what an auditor or an acquirer's due-diligence review wants to see.

### AI-generated content is labeled

Where WhiteBoxXAI drafts content for a board — review templates, best-practice suggestions,
decision summaries — that output is labeled as AI-generated pending human sign-off. An
approval is never allowed to look like a human judgement when it was substantively
AI-authored.

## Submit and vote on a request

Optionally start from an AI-generated template, then create the request:

```bash
# 1. Generate an AI review template
POST /api/v1/governance/review-boards/ai/generate-template
{
  "review_type": "model_deployment",
  "board_id": "...",
  "context": { "model_name": "Fraud Detection v3.0", "model_version": "3.0.1" }
}

# 2. Create the request
POST /api/v1/governance/review-boards/requests
{
  "board_id": "...",
  "title": "Deploy Fraud Detection v3.0",
  "description": "<review details>",
  "request_type": "model_deployment",
  "priority": "high"
}
```

Submit the request to open it for voting, or withdraw it if it's no longer needed:

```bash
POST /api/v1/governance/review-boards/requests/{request_id}/submit
POST /api/v1/governance/review-boards/requests/{request_id}/withdraw
```

Members cast votes:

```bash
POST /api/v1/governance/review-boards/requests/{request_id}/decisions
{
  "decision_type": "approve",
  "rationale": "Model meets all compliance requirements."
}
```

A vote from the request's own submitter is rejected — see [Separation of
duties](#separation-of-duties).

Check status at any time:

```bash
GET /api/v1/governance/review-boards/requests/{request_id}/status
# -> total_members, votes_cast, approve_count, threshold_met, quorum_met, can_finalize
```

Board members can see what's waiting on them:

```bash
GET /api/v1/governance/review-boards/my-reviews
GET /api/v1/governance/review-boards/{board_id}/dashboard
```

In the dashboard, these are **My Requests** and the board's own overview page.

A request can be finalized once both **quorum** and the **approval threshold** are met.
Finalizing creates an archive entry with an AI-generated executive summary:

```bash
POST /api/v1/governance/review-boards/requests/{request_id}/finalize
```

!!! note "A denied review becomes a tracked risk"
    Denying a request, or sending it back with **request changes**, automatically drafts an
    entry in your [AI Risk Register](risk-register.md) so the concern is owned and tracked
    rather than left in a vote record. See [Automatic risk
    drafting](risk-register.md#automatic-risk-drafting).

## AI-assisted governance

WhiteBoxXAI can help boards work faster and more consistently:

- **Template generation** — structured review templates based on the review type, model
  context, and board type.
- **Best-practice suggestions** — recommendations that reference the NIST AI Risk
  Management Framework, ISO/IEC 42001, and the EU AI Act.
- **Decision summaries** — an executive summary, key concerns, conditions of approval, and
  searchable keywords, generated automatically when a decision is finalized.

## Automated periodic reviews

Schedule recurring reviews so high-risk models are re-assessed on a cadence:

```bash
POST /api/v1/governance/review-boards/schedules
{
  "board_id": "...",
  "schedule_name": "Quarterly Risk Assessment",
  "cron_expression": "0 9 1 */3 *",
  "review_type": "risk_assessment",
  "ai_agent_enabled": true,
  "target_criteria": { "status": "ACTIVE", "tags": ["high-risk"] }
}
```

**Cron examples**

```text
0 9 * * *          Every day at 9 AM
0 9 1 * *          First day of every month
0 14 * * 1         Every Monday at 2 PM
0 9 1 1,4,7,10 *   Quarterly (Jan, Apr, Jul, Oct)
```

## Search the decision archive

Every finalized decision is stored and full-text searchable, with faceted filters and CSV
export for audits. In the dashboard this is **Decisions Archive**.

```bash
POST /api/v1/governance/review-boards/archive/search
{
  "query": "fraud detection bias",
  "final_decision": "approved",
  "date_from": "2024-01-01",
  "keywords": ["fairness", "bias"],
  "limit": 20
}
```

You can filter by final decision, request type, date range, and keywords, then export the
results for compliance reporting or historical analysis.

Other archive endpoints:

```bash
GET  /api/v1/governance/review-boards/archive/recent      # Most recent decisions
GET  /api/v1/governance/review-boards/archive/facets       # Available filter values
GET  /api/v1/governance/review-boards/archive/{decision_id}
POST /api/v1/governance/review-boards/archive/export       # CSV export for audits
```

### Corrections

The archive is append-only — see [The decision archive is
immutable](#the-decision-archive-is-immutable). To correct a decision recorded in error,
record a superseding entry rather than editing the original. Both remain in the archive,
linked by `supersedes_id`, with the `correction_reason` explaining why.

When searching, be aware that a request can therefore have more than one archive entry: the
original and any corrections that supersede it.

## Security & isolation

- All governance data is scoped to your organization — there is no cross-organization
  access.
- Separation of duties, vote history, and archive immutability are enforced at the database
  layer, not only in application code.
- API keys are stored hashed and are never returned in API responses after creation. See
  [API Keys](../account/api-keys.md).

## Troubleshooting

**AI templates aren't generating** — confirm AI features are enabled for your organization
in settings.

**Votes aren't counting toward the threshold** — check that voters are active board
members (not observers), that `voting_power` is set, and that `quorum_required` isn't
higher than your member count.

**A vote is rejected outright** — the voter is the request's own submitter. [Separation of
duties](#separation-of-duties) blocks self-voting, so the request needs a different voter,
or a different submitter.

**A decision isn't in the archive** — the request must be **finalized** to create an
archive entry.

**A wrong decision can't be edited** — that's intentional. Record a
[correction](#corrections) that supersedes it instead.

**Scheduled reviews aren't running** — verify the schedule is active, its next run time
has passed, and its `target_criteria` matches existing models.
