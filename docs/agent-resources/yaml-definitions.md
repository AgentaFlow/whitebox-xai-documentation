---
description: Copy-paste MCP client configurations (JSON and YAML) for connecting Claude Desktop, Claude Code, Cursor, VS Code and other agents to the WhiteBoxXAI MCP server, plus an n8n webhook node definition.
---

# Agent Config Definitions

Machine-readable configuration for connecting agents to WhiteBoxXAI. Replace `wbx_live_...` with an
API key from **Settings → API keys** in the dashboard, and use your client's secret or input
mechanism for it where one exists. Don't commit the key.

## MCP server: local (stdio)

The `whiteboxxai-mcp` command runs the server on your machine and talks to WhiteBoxXAI with your API
key. Install it with `pip install whiteboxxai-mcp` (Python 3.11 or later).

Most MCP clients (Claude Desktop, Cursor, Windsurf) use this JSON shape:

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

The same definition as YAML, for harnesses that take YAML:

```yaml
mcpServers:
  whiteboxxai:
    command: whiteboxxai-mcp
    env:
      WHITEBOXXAI_MCP_API_BASE_URL: https://api.whiteboxxai.com
      WHITEBOXXAI_MCP_API_KEY: ${WHITEBOXXAI_MCP_API_KEY}
```

*Download: [whiteboxxai-mcp.yaml](../assets/downloads/whiteboxxai-mcp.yaml)*

Client-specific variants (VS Code's `servers` key, the Claude Code CLI, remote HTTP) are on the
[MCP Server](../integrations/mcp.md#connect-a-client) page.

## MCP server: hosted (Streamable HTTP)

Clients that support remote MCP servers can connect to WhiteBoxXAI's hosted endpoint without
installing anything. Every request needs an `Authorization` header.

```json
{
  "mcpServers": {
    "whiteboxxai": {
      "type": "http",
      "url": "https://api.whiteboxxai.com/mcp/",
      "headers": {
        "Authorization": "Bearer wbx_live_..."
      }
    }
  }
}
```

## n8n webhook node

A starting point for an n8n workflow that receives WhiteBoxXAI alert notifications on a webhook.
This is a single n8n **Webhook node** definition shown in YAML for readability; n8n imports
workflows as JSON, so convert it (or build the node in the n8n editor with the same settings).
Point a WhiteBoxXAI webhook notification channel at the URL n8n gives you. See
[n8n Webhooks](../integrations/n8n-webhooks.md) for sending n8n execution data *to* WhiteBoxXAI.

```yaml
name: WhiteBoxXAI Alert Listener
nodes:
  - name: Webhook
    type: n8n-nodes-base.webhook
    typeVersion: 1
    position: [250, 300]
    parameters:
      httpMethod: POST
      path: whitebox-alerts
      responseMode: lastNode
      options: {}
```

*Download: [n8n-alert-webhook.yaml](../assets/downloads/n8n-alert-webhook.yaml)*
