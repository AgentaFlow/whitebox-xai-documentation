# Agent YAML Definitions

This page provides machine-readable YAML definitions used to configure agents and system integrations within the WhiteBoxXAI ecosystem.

## MCP Server Manifest (`whiteboxxai-mcp.yaml`)

Use this manifest to connect your AI agent to the WhiteBoxXAI Model Context Protocol (MCP) server. 

```yaml
mcpServers:
  whiteboxxai:
    command: "python"
    args:
      - "-m"
      - "whiteboxxai_mcp"
    env:
      WHITEBOX_API_KEY: "${WHITEBOX_API_KEY}"
```
*Download: [whiteboxxai-mcp.yaml](../assets/downloads/whiteboxxai-mcp.yaml)*

## n8n Webhook Configuration (`n8n-alert-webhook.yaml`)

Use this configuration to scaffold an n8n webhook workflow that listens to WhiteBoxXAI alerts.

```yaml
name: WhiteBoxXAI Alert Listener
nodes:
  - parameters:
      httpMethod: POST
      path: whitebox-alerts
      responseMode: lastNode
      options: {}
    name: Webhook
    type: n8n-nodes-base.webhook
    typeVersion: 1
    position:
      - 250
      - 300
    webhookId: whitebox-alerts-123
```
*Download: [n8n-alert-webhook.yaml](../assets/downloads/n8n-alert-webhook.yaml)*
