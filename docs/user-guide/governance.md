---
description: WhiteBoxXAI Reviews & Decisions — governance review boards, review requests, votes signed with an authenticator code, separation of duties, the immutable decision archive, RACI assignments and scheduled reviews.
---

# Reviews & Decisions

Governance Review Boards bring executive oversight to your AI models through multi-party
approval workflows, AI-assisted review templates, a searchable decision archive, and
automated periodic reviews. Use them to make sure the right people sign off before a model
goes to production — and to keep an auditable record of every decision.

In the [Enhanced Dashboard](index.md) this is **Reviews & Decisions** (`g` then `v`), with five tabs:

| Tab | Shows |
| --- | --- |
| **My reviews** | Requests waiting for *your* decision |
| **Requests** | Every request, filterable by status (Draft, Submitted, Under review, Approved, Denied, Withdrawn, Archived) and type |
| **Boards** | Your review boards, with their **Members**, **Rules**, **Activity** and **Schedules** |
| **Decisions** | The searchable decision archive, with **Export CSV** (up to 10,000 rows) |
| **RACI** | Who's Responsible, Accountable, Consulted and Informed, per board |

You can also automate most of it through the REST API. API examples below use the base URL
`https://api.whiteboxxai.com` and an [API key](../account/api-keys.md)
(`Authorization: Bearer $WHITEBOX_API_KEY`). **Voting and finalizing are the exception:** they need
a person's own signed-in session and an authenticator code. See
[Vote with an authenticator code](#vote-with-an-authenticator-code).

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

In the dashboard, an administrator goes to **Reviews & Decisions → Boards → New board**, or uses the API:

```bash
curl -X POST https://api.whiteboxxai.com/api/v1/governance/review-boards \
  -H "Authorization: Bearer $WHITEBOX_API_KEY" \
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

**A request's submitter cannot vote on their own request.** This is enforced by the platform on
every write path, so it can't be circumvented by calling the API directly rather than using
the dashboard. A member can also only cast one
vote per request.

This is the property that makes a review board *governance* rather than an approval button —
and it's why a "zero self-approvals" statement about your organization is provable rather
than asserted.

### Vote changes are recorded, never overwritten

A member may change their vote while a request is still open. When they do, the previous vote
is preserved in an append-only vote history rather than being replaced. Nothing about a
vote's history is silently lost.

Once a request is finalized, votes are closed.

### The decision archive is immutable

Finalized decisions cannot be edited or deleted — edits and deletions are refused for both
the archive and the vote history, including through the API.

If a decision was recorded in error, you don't mutate history. You record a **correction**: a
new archive entry that supersedes the original, carrying a reference to it (`supersedes_id`)
and a `correction_reason`. Both entries remain, and the chain between them is explicit — which
is exactly what an auditor wants to see.

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

Members cast votes from the request's slide-over in **Reviews & Decisions**: **Approve**, **Deny**,
**Abstain** or **Request changes**, with a **Justification** and optional **Conditions**. The panel
shows the approval weight so far (**Approval weight: X of Y needed**) and whether quorum is met.

### Vote with an authenticator code

A vote is a sign-off that ends up in an immutable archive, so it has to be provably *yours* and made
against evidence you actually saw:

- **Two-factor authentication must be on.** Without it, the vote is refused with *"Voting and
  sign-off need two-factor authentication. Turn it on in Settings > Security, then try again."*
- **Each vote and each finalization needs a current authenticator code** (or one of your backup
  codes), entered in the **Authenticator code** field. Repeated wrong codes lock voting for a while
  and are recorded in the audit trail.
- **It must be your own session.** Organization API keys can't vote or finalize, so neither can an
  automated pipeline or an AI agent acting with a key.
- **The evidence must not have changed.** Every request has an evidence snapshot hash. A vote
  carries the hash of the version you reviewed, and if the request changed after you opened it, the
  vote is refused: *"This request changed after you opened it. Reload it and review the current
  version before voting."*

Over the API, from a person's own session:

```bash
POST /api/v1/governance/review-boards/requests/{request_id}/decisions
{
  "decision": "approve",
  "justification": "Model meets all compliance requirements.",
  "conditions": "Re-run the fairness audit after the next retrain.",
  "evidence_snapshot_hash": "<from the request detail you reviewed>",
  "totp_code": "123456"
}
```

AI assistants can prepare and summarise a request's dossier for you through the
[MCP server](../integrations/mcp.md#governance-voting), and in hosts that support interactive cards
can show you voting buttons, but the vote is still yours, made with your session and your code.

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

In the dashboard, these are the **My reviews** tab and the board's slide-over on the **Boards** tab.

A request can be finalized once both **quorum** and the **approval threshold** are met, by an
administrator or the board's chair. Finalizing is the formal sign-off, so it needs an authenticator
code too. It creates an archive entry with an AI-generated executive summary:

```bash
POST /api/v1/governance/review-boards/requests/{request_id}/finalize
{ "totp_code": "123456" }
```

!!! note "A denied review becomes a tracked risk"
    Denying a request, or sending it back with **request changes**, automatically drafts an
    entry in your [AI Risk Register](risk-register.md) so the concern is owned and tracked
    rather than left in a vote record. See [Automatic risk
    drafting](risk-register.md#automatic-risk-drafting).

## Requests in the dashboard

**Requests → Raise a review request** opens the request form. **Generate AI Template** drafts the
description from the request type and system, and AI-drafted text stays marked *"AI-generated
content — pending human sign-off"* until a person edits it. Choose **Save as draft** or **Save and
submit**.

A request's slide-over has everything a reviewer needs in one place:

- **Submit to the board**, **Withdraw** and **Finalize the decision**
- **Votes**, with approval weight and quorum (see [voting](#vote-with-an-authenticator-code))
- **Discussion**: comments typed as Discussion, Question, Concern, Suggestion, Approval note or
  Denial reason, with replies
- **Attachments**: PDF, PNG, JPG, CSV, Markdown or text files, such as an architecture diagram or a
  test report. The requester, board members and administrators can attach files until a decision
  is made.
- **RACI** assignments
- **AI Governance Suggestions** and **AI Best Practices** for the request type

### Boards

On the **Boards** tab, administrators create boards (**New board**), add members with a role,
voting power, department and expertise, and edit the rules: quorum, approval weight, the escalation
board and how often reviews recur. Boards can be deactivated and reactivated. Everyone can see a
board's **Members**, **Rules**, **Activity** and **Schedules**.

## RACI assignment

Beyond a vote, a review request can carry a RACI assignment — who's Responsible, Accountable,
Consulted, and Informed — the structure ISO/IEC 42001 Clause 5.3 and Annex A.3 expect for AI
governance roles and responsibilities.

In the dashboard, this is the **RACI** tab of Reviews & Decisions: a per-board matrix of requests
against members, with R, A, C and I assignments drawn from the board's real membership. Assign roles
from a request's slide-over.

```bash
GET  /api/v1/governance/review-boards/raci-grid       # the full grid, optionally filtered to one board
POST /api/v1/governance/review-boards/requests/{request_id}/raci
GET|PATCH|DELETE /api/v1/governance/review-boards/requests/{request_id}/raci/{assignment_id}
```

The [`whitebox_governance_raci_grid`](../integrations/mcp.md#governance) MCP tool exposes the
same grid read-only for agent clients.

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
export for audits. In the dashboard this is the **Decisions** tab: search (use quotes for a phrase
and `-word` to exclude), filter by outcome, type, board and date, and **Export CSV**.

```bash
POST /api/v1/governance/review-boards/archive/search
{
  "query": "fraud detection bias",
  "final_decision": "approved",
  "date_from": "2026-01-01",
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
- Separation of duties, vote history, and archive immutability are enforced on every write path,
  including direct API calls, not only in the dashboard.
- Votes and finalization need the voter's own session and a current authenticator code.
- API keys are stored hashed and are never returned in API responses after creation. See
  [API Keys](../account/api-keys.md).

## Troubleshooting

**AI templates aren't generating** — confirm AI features are enabled for your organization
in settings.

**Votes aren't counting toward the threshold** — check that voters are active board
members (not observers), that `voting_power` is set, and that `quorum_required` isn't
higher than your member count.

**A vote is refused with 403 asking for two-factor authentication**: turn on two-factor in
**Settings → Profile & security**. Organization API keys can't vote at all.

**A vote is refused with 409**: the request changed after you opened it. Reload, review the current
version, and vote again.

**A vote is rejected outright**: the voter is the request's own submitter. [Separation of
duties](#separation-of-duties) blocks self-voting, so the request needs a different voter,
or a different submitter.

**A decision isn't in the archive** — the request must be **finalized** to create an
archive entry.

**A wrong decision can't be edited** — that's intentional. Record a
[correction](#corrections) that supersedes it instead.

**Scheduled reviews aren't running** — verify the schedule is active, its next run time
has passed, and its `target_criteria` matches existing models.
