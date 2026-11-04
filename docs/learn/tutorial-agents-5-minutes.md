---
description: Five-minute quickstart — install whiteboxxai-mcp with pip, connect your AI assistant to WhiteBoxXAI, load the governance rules, and get a first answer and a first drift record that came from real data.
---

# Govern your agents in 5 minutes

You'll connect an AI assistant to WhiteBoxXAI over the [MCP server](../integrations/mcp.md), tell it the
rules it must follow, and check that its first answer and its first record come from your data. For the
longer version, with compliance questions and a governance review, see
[Connect an AI assistant over MCP](tutorial-agent-mcp.md).

**You need:** an MCP-capable client (Claude Desktop, Claude Code, Cursor or VS Code), Python 3.11 or
later, and a WhiteBoxXAI [API key](../account/api-keys.md) (administrators create these in
**Settings → API keys**).

## 1. Install

```bash
pip install whiteboxxai-mcp
```

## 2. Connect your client

For Claude Desktop (`claude_desktop_config.json`) or Cursor (`~/.cursor/mcp.json`), add:

```json
{
  "mcpServers": {
    "whiteboxxai": {
      "command": "whiteboxxai-mcp",
      "env": {
        "WHITEBOXXAI_MCP_API_BASE_URL": "https://api.whiteboxxai.com",
        "WHITEBOXXAI_MCP_API_KEY": "wbx_live_..."
      }
    }
  }
}
```

For Claude Code:

```bash
claude mcp add whiteboxxai \
  --env WHITEBOXXAI_MCP_API_KEY=wbx_live_... \
  -- whiteboxxai-mcp
```

`WHITEBOXXAI_MCP_API_BASE_URL` defaults to `https://api.whiteboxxai.com`. Business Cloud tenants set it to
the API address they were given. Restart the client so it starts the server. Other clients are on the
[MCP Server](../integrations/mcp.md#connect-a-client) page.

## 3. Give the assistant its rules

Tell the assistant:

```text
Read https://whiteboxxai.com/.well-known/skills/whitebox-governance-rules/SKILL.md and
https://whiteboxxai.com/.well-known/skills/whitebox-mcp-server/SKILL.md and follow them.
```

The rules tell it to report empty or unassessed results as they are, never invent a number, and never
cast a governance vote. See [Agent Skills](../agent-resources/skill-libraries.md).

## 4. Check that it works

Ask:

```text
Which AI systems do we govern?
```

The assistant should call `whitebox_models_list` and answer with the names, versions and statuses it
returned. If your account has no systems yet, a correct answer says so; it doesn't make some up. Load
[sample data](interactive-demos.md) if you want something to look at.

Then have it leave a record:

```text
Has the first model on that list drifted? Create a drift report so we have a record, and give me its id.
```

The assistant should call `whitebox_drift_detect`, then `whitebox_drift_create_report`, and quote the
report id. Find that report in the dashboard to confirm it exists: that's your check that the assistant
reported what it did rather than what it expected to do.

## What you've set up

- An assistant that reads and records through the MCP Governance Layer, on your API key
- Rules that keep it from inventing numbers or voting on your behalf
- A drift report you can open in the dashboard and show to a reviewer

Revoke the API key in **Settings → API keys** when you're done experimenting. To go further, follow the
[20-minute tutorial](tutorial-agent-mcp.md): compliance questions, alert triage, and preparing a review
that a person decides.
