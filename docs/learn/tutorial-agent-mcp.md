---
description: Hands-on tutorial — connect Claude Desktop, Claude Code, Cursor or VS Code to WhiteBoxXAI over MCP, load the agent skills, ask compliance questions, and have the assistant prepare a governance review for a person to decide.
---

# Tutorial: connect an AI assistant over MCP

You'll give an AI assistant read-and-act access to your WhiteBoxXAI data through the
[MCP server](../integrations/mcp.md), and use it the way a risk officer would: *where do we stand on
ISO 42001, what's firing, what's waiting for review?* It takes about 20 minutes and needs no code.

**You need:** an MCP-capable client (Claude Desktop, Claude Code, Cursor or VS Code), a WhiteBoxXAI
[API key](../account/api-keys.md), and Python 3.11 or later for the local server. Some data helps: load
[sample data](interactive-demos.md) if your account is empty.

## 1. Install and connect

```bash
pip install whiteboxxai-mcp
```

Add the server to your client. For Claude Desktop (`claude_desktop_config.json`) or Cursor
(`~/.cursor/mcp.json`):

```json
{
  "mcpServers": {
    "whiteboxxai": {
      "command": "whiteboxxai-mcp",
      "env": {
        "WHITEBOXXAI_MCP_API_KEY": "wbx_live_..."
      }
    }
  }
}
```

For Claude Code:

```bash
claude mcp add whiteboxxai --env WHITEBOXXAI_MCP_API_KEY=wbx_live_... -- whiteboxxai-mcp
```

Restart the client. It should list the WhiteBoxXAI tools (197 of them). Other clients are covered on
the [MCP Server](../integrations/mcp.md#connect-a-client) page.

## 2. Give the assistant its rules

Paste the [compliance-assistant prompt](../agent-resources/prompts.md#compliance-assistant) into your
client's system or project instructions, or tell the assistant:

```text
Read https://whiteboxxai.com/.well-known/skills/whitebox-governance-rules/SKILL.md and
https://whiteboxxai.com/.well-known/skills/whitebox-compliance-evidence/SKILL.md and follow them.
```

The skills tell the assistant to report empty or unassessed results as they are, never invent a
number, and never cast a governance vote. See [Agent Skills](../agent-resources/skill-libraries.md).

## 3. Ask compliance questions

Try these, and watch which tools the assistant calls:

| Ask | Expect |
| --- | --- |
| "Which AI systems do we govern?" | `whitebox_models_list`, with names, versions and statuses |
| "Where are we on ISO 42001? Lead with the verdict." | `whitebox_compliance_iso42001_status`; in clients that support MCP Apps, an ISO/IEC 42001 status card |
| "What are our open high-severity ISO 42001 gaps?" | `whitebox_compliance_iso42001_gaps` |
| "What's firing right now, and why?" | `whitebox_alerts_list_instances`, then `whitebox_alerts_get_triage_context` for the top alert |
| "Show me the risk heat map and the top open risks." | `whitebox_risk_register_portfolio` |

Check that every number in an answer came from a tool result, and that unassessed controls are called
"not yet assessed" rather than given a score.

## 4. Have it keep a record

```text
Has credit_scoring drifted? Create a drift report so we have a record, and give me its id.
```

The assistant should call `whitebox_drift_detect`, then `whitebox_drift_create_report`, and quote the
report id. The report now appears in the system's dossier in the dashboard.

## 5. Prepare a review, and decide it yourself

```text
What's waiting for review? Summarise the oldest request for me: what changed, what evidence exists,
what's missing, and the evidence snapshot hash.
```

The assistant uses `whitebox_governance_list_review_requests` and
`whitebox_governance_get_review_request`. In clients that support MCP Apps, the governance dossier card
shows the request with voting buttons.

The vote itself stays with you. Voting needs your own signed-in session on the hosted endpoint,
two-factor authentication and an authenticator code, so an assistant running on an API key can't vote.
Vote in the card from a personal session, or in **Reviews & Decisions** in the dashboard. See
[Governance voting](../integrations/mcp.md#governance-voting).

## What you've set up

- An assistant that reads your WhiteBoxXAI data through the MCP Governance Layer
- Guardrails from the skills: no invented numbers, verdicts with evidence, no automated votes
- A drift report created on request and visible to everyone in the dashboard

Revoke the API key in **Settings → API keys** when you're done experimenting.
