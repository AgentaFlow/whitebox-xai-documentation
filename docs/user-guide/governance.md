# Governance Review Boards

Governance Review Boards bring executive oversight to your AI models through multi-party
approval workflows, AI-assisted review templates, a searchable decision archive, and
automated periodic reviews. Use them to make sure the right people sign off before a model
goes to production — and to keep an auditable record of every decision.

You can manage everything from the **Governance** area of the dashboard, or automate it
through the REST API. API examples below use the base URL `https://api.whiteboxxai.com`
and require an API key (`Authorization: Bearer YOUR_TOKEN`).

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

Members cast votes:

```bash
POST /api/v1/governance/review-boards/requests/{request_id}/decisions
{
  "decision_type": "approve",
  "rationale": "Model meets all compliance requirements."
}
```

Check status at any time:

```bash
GET /api/v1/governance/review-boards/requests/{request_id}/status
# -> total_members, votes_cast, approve_count, threshold_met, quorum_met, can_finalize
```

A request can be finalized once both **quorum** and the **approval threshold** are met.
Finalizing creates an archive entry with an AI-generated executive summary:

```bash
POST /api/v1/governance/review-boards/requests/{request_id}/finalize
```

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
export for audits:

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

## Security & isolation

- All governance data is scoped to your organization — there is no cross-organization
  access.
- API keys are stored encrypted and are never returned in API responses.

## Troubleshooting

**AI templates aren't generating** — confirm AI features are enabled for your organization
in settings.

**Votes aren't counting toward the threshold** — check that voters are active board
members (not observers), that `voting_power` is set, and that `quorum_required` isn't
higher than your member count.

**A decision isn't in the archive** — the request must be **finalized** to create an
archive entry.

**Scheduled reviews aren't running** — verify the schedule is active, its next run time
has passed, and its `target_criteria` matches existing models.
