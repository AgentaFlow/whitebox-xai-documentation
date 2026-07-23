# n8n Webhook Integration Guide

## Overview

The WhiteBoxXAI n8n integration enables monitoring of n8n workflow automation through webhooks. This provides observability into your n8n workflows, tracking node executions, AI agent interactions, and workflow performance.

### Supported Features

- **Workflow Monitoring**: Track complete n8n workflow executions from start to finish
- **Node-Level Tracking**: Monitor individual node executions with timing and status
- **AI Node Detection**: Automatically identify and track AI-powered nodes (OpenAI, AI Agent, etc.)
- **Error Tracking**: Capture and log node failures with detailed error information
- **Multi-Tenancy**: Organization-scoped workflow tracking with proper isolation
- **Idempotency**: Handle duplicate webhook calls gracefully

### Architecture

```
┌─────────────┐         ┌──────────────────┐         ┌─────────────────┐
│  n8n Cloud  │ Webhook │  WhiteBoxXAI API  │  Maps   │   WhiteBoxXAI    │
│     or      ├────────►│   /webhooks/n8n  ├────────►│   Database      │
│ Self-Hosted │  POST   │   /execution     │  Nodes  │  (Workflows)    │
└─────────────┘         └──────────────────┘  to     └─────────────────┘
                                               Agents
```

**Mapping Logic:**
- **n8n Workflow** → WhiteBoxXAI Workflow
- **n8n Node** → WhiteBoxXAI Agent
- **Node Execution** → Agent Execution
- **Workflow Run** → Workflow Lifecycle (create → start → complete)

---

## Quick Start

### 1. Get Webhook Configuration

```bash
curl -X GET "https://api.whiteboxxai.com/api/v1/webhooks/n8n/config" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Response:
```json
{
  "webhook_url": "https://api.whiteboxxai.com/api/v1/webhooks/n8n/execution",
  "api_key": "wbai_...",
  "track_all_nodes": true,
  "min_execution_time_ms": 0
}
```

### 2. Configure n8n Webhook Node

Add a **Webhook** node to your n8n workflow (at the end to capture completion):

**Webhook Configuration:**
```json
{
  "httpMethod": "POST",
  "path": "",
  "responseMode": "onReceived",
  "options": {},
  "authentication": "headerAuth",
  "headerAuth": {
    "name": "Authorization",
    "value": "Bearer YOUR_WhiteBoxXAI_API_KEY"
  }
}
```

**Set Webhook URL:**
```
https://api.whiteboxxai.com/api/v1/webhooks/n8n/execution
```

### 3. Configure Workflow Settings

In your n8n workflow settings, add a **Webhook** node and configure the payload:

**Expression to Send (in HTTP Request body):**
```javascript
{
  "workflow_id": "{{ $workflow.id }}",
  "workflow_name": "{{ $workflow.name }}",
  "execution_id": "{{ $execution.id }}",
  "status": "{{ $execution.mode === 'manual' && $execution.finished ? 'success' : 'error' }}",
  "started_at": "{{ $execution.startedAt }}",
  "finished_at": "{{ $now }}",
  "mode": "{{ $execution.mode }}",
  "nodes": {{ $json.nodes }},
  "meta_data": {
    "project": "your-project-name",
    "environment": "production"
  }
}
```

### 4. Collect Node Data

To send node execution data, add a **Code** node before the webhook:

```javascript
// Collect all node execution data
const nodes = [];

for (const itemIndex in items) {
  const item = items[itemIndex];

  // Get all previous nodes
  for (const nodeName of Object.keys($input.all())) {
    const nodeData = $input.all()[nodeName];

    if (nodeData && nodeData.length > 0) {
      nodes.push({
        node_name: nodeName,
        node_type: nodeData[0].json.nodeType || "unknown",
        input_data: nodeData[0].json.input || {},
        output_data: nodeData[0].json,
        execution_time_ms: nodeData[0].json.executionTime || 0,
        status: "success"
      });
    }
  }
}

return [{ json: { nodes } }];
```

---

## Complete Example

### Example 1: AI Content Generation Workflow

This n8n workflow uses OpenAI to generate blog content, then checks grammar:

**Workflow Structure:**
1. **Trigger** (Manual or Schedule)
2. **OpenAI Agent** - Generate blog post
3. **Grammar Check** - Validate content
4. **Collect Data** (Code node)
5. **Send to WhiteBoxXAI** (Webhook node)

**Code Node (Step 4):**
```javascript
const workflowId = $workflow.id;
const workflowName = $workflow.name;
const executionId = $execution.id;
const startedAt = new Date($execution.startedAt).toISOString();
const finishedAt = new Date().toISOString();

// Collect node executions
const nodes = [
  {
    node_name: "OpenAI Agent",
    node_type: "n8n-nodes-base.openAi",
    input_data: {
      prompt: "Write a blog post about AI in healthcare"
    },
    output_data: $node["OpenAI Agent"].json,
    execution_time_ms: 2500,
    status: "success"
  },
  {
    node_name: "Grammar Check",
    node_type: "n8n-nodes-base.httpRequest",
    input_data: {
      text: $node["OpenAI Agent"].json.text
    },
    output_data: $node["Grammar Check"].json,
    execution_time_ms: 1200,
    status: "success"
  }
];

return [{
  json: {
    workflow_id: workflowId,
    workflow_name: workflowName,
    execution_id: executionId,
    status: "success",
    started_at: startedAt,
    finished_at: finishedAt,
    mode: $execution.mode,
    nodes: nodes,
    meta_data: {
      project: "content-automation",
      version: "1.0"
    }
  }
}];
```

**Webhook Node (Step 5):**
- **URL**: `https://api.whiteboxxai.com/api/v1/webhooks/n8n/execution`
- **Method**: POST
- **Authentication**: Bearer Token (WhiteBoxXAI API Key)
- **Body**: `{{ $json }}`

### Example 2: Error Handling

Handle failed nodes in your workflow:

```javascript
// In Code node - detect failures
const hasError = $execution.mode === 'manual' &&
                 !$execution.finished;

const status = hasError ? "error" : "success";

const nodes = [];
for (const nodeName of Object.keys($input.all())) {
  const nodeData = $input.all()[nodeName];

  // Check if node failed
  const nodeStatus = nodeData[0].json.error ? "error" : "success";
  const error = nodeData[0].json.error || null;

  nodes.push({
    node_name: nodeName,
    node_type: nodeData[0].json.nodeType || "unknown",
    input_data: nodeData[0].json.input || {},
    output_data: nodeData[0].json.error ? {} : nodeData[0].json,
    execution_time_ms: nodeData[0].json.executionTime || 0,
    status: nodeStatus,
    error: error
  });
}

return [{
  json: {
    workflow_id: $workflow.id,
    workflow_name: $workflow.name,
    execution_id: $execution.id,
    status: status,
    started_at: new Date($execution.startedAt).toISOString(),
    finished_at: new Date().toISOString(),
    mode: $execution.mode,
    nodes: nodes
  }
}];
```

---

## Webhook Payload Reference

### Request Schema

**Endpoint:** `POST /api/v1/webhooks/n8n/execution`

**Headers:**
```
Authorization: Bearer YOUR_API_KEY
Content-Type: application/json
```

**Payload:**
```json
{
  "workflow_id": "string",
  "workflow_name": "string",
  "execution_id": "string",
  "status": "success|error|running",
  "started_at": "2026-01-28T10:00:00Z",
  "finished_at": "2026-01-28T10:02:30Z",
  "mode": "manual|trigger|webhook|production",
  "nodes": [
    {
      "node_name": "string",
      "node_type": "string",
      "input_data": {},
      "output_data": {},
      "execution_time_ms": 0,
      "status": "success|error|waiting",
      "error": "string (optional)"
    }
  ],
  "meta_data": {}
}
```

### Response Schema

```json
{
  "status": "success|error",
  "workflow_id": "uuid",
  "message": "string",
  "received_at": "2026-01-28T10:02:31Z"
}
```

---

## Node Type Detection

WhiteBoxXAI automatically detects AI nodes based on node type:

**AI Nodes (tracked as AI agents):**
- `n8n-nodes-base.openAi`
- `n8n-nodes-base.openAiChat`
- `*ai*` (any node type containing "ai")
- Custom AI nodes

**Other Nodes (tracked as utility agents):**
- HTTP Request nodes
- Code nodes
- Webhook nodes
- Database nodes
- All other n8n nodes

---

## Advanced Features

### 1. Custom Metadata

Add custom metadata to track workflow context:

```javascript
{
  // ... standard fields ...
  "meta_data": {
    "project": "content-automation",
    "environment": "production",
    "customer_id": "cust_12345",
    "pipeline_stage": "draft",
    "cost_center": "marketing"
  }
}
```

### 2. Duplicate Execution Handling

WhiteBoxXAI automatically handles duplicate webhook calls:

- Checks for existing workflows with same `n8n_execution_id` in metadata
- Updates existing workflow instead of creating duplicate
- Idempotent operations ensure data consistency

### 3. Workflow Tracking

Each n8n execution is tracked with:

```
External ID: n8n:{workflow_id}:{execution_id}
Framework: n8n
Status: running → completed/failed
```

### 4. Agent Registration

Nodes are automatically registered as agents:

```
Name: {node_name}
Role: {node_type} (e.g., "OpenAI Agent")
Is AI: true (for AI nodes)
```

### 5. Execution Logging

Each node execution is logged with:

- Input/output data
- Execution time (ms)
- Status (success/error/waiting)
- Error messages (if failed)

---

## Testing

### Test Webhook Endpoint

```bash
curl -X POST "https://api.whiteboxxai.com/api/v1/webhooks/n8n/test" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json"
```

Response:
```json
{
  "status": "test_successful",
  "workflow_id": "uuid",
  "message": "Test webhook processed successfully"
}
```

### Manual Test with Sample Data

```bash
curl -X POST "https://api.whiteboxxai.com/api/v1/webhooks/n8n/execution" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "workflow_id": "wf_test_123",
    "workflow_name": "Test Workflow",
    "execution_id": "exec_test_456",
    "status": "success",
    "started_at": "2026-01-28T10:00:00Z",
    "finished_at": "2026-01-28T10:01:00Z",
    "mode": "manual",
    "nodes": [
      {
        "node_name": "Test Node",
        "node_type": "n8n-nodes-base.test",
        "input_data": {"test": "data"},
        "output_data": {"result": "success"},
        "execution_time_ms": 1000,
        "status": "success"
      }
    ]
  }'
```

---

## Best Practices

### 1. Webhook Placement

**Recommended:** Place webhook node at the **end** of your workflow to capture complete execution data.

```
[Trigger] → [Node 1] → [Node 2] → [Collect Data] → [WhiteBoxXAI Webhook]
```

### 2. Error Handling

Always wrap webhook calls in try-catch to prevent workflow failures:

```javascript
try {
  // Collect and send data
  const webhookData = { /* ... */ };
  // Send to WhiteBoxXAI
} catch (error) {
  console.error("Failed to send to WhiteBoxXAI:", error);
  // Continue workflow execution
}
```

### 3. Data Collection

Use a dedicated **Code** node to collect all node execution data before sending to webhook.

### 4. Authentication

- Use **Bearer Token** authentication
- Store API key in n8n credentials (encrypted)
- Never hardcode API keys in workflow JSON

### 5. Rate Limiting

- WhiteBoxXAI has no rate limits for webhooks
- However, batch operations where possible
- Use async webhook mode (`responseMode: "onReceived"`)

### 6. Metadata

Include relevant metadata for filtering and analysis:

```javascript
{
  "meta_data": {
    "project": "project-name",
    "environment": "production|staging|dev",
    "version": "1.0.0",
    "tags": ["ai", "automation", "content"]
  }
}
```

---

## Troubleshooting

### Issue 1: Webhook Not Receiving Data

**Symptoms:** No workflows appearing in WhiteBoxXAI dashboard

**Solutions:**
1. Verify webhook URL is correct
2. Check API key is valid and has correct permissions
3. Verify n8n workflow is actually executing
4. Check n8n webhook node logs for errors
5. Test webhook endpoint manually (see Testing section)

### Issue 2: Incomplete Node Data

**Symptoms:** Some nodes not tracked in WhiteBoxXAI

**Solutions:**
1. Ensure **Code** node collects all previous nodes
2. Check node execution order
3. Verify all nodes complete before webhook fires
4. Use `$input.all()` to get all node data

### Issue 3: Duplicate Workflows

**Symptoms:** Same execution creating multiple workflows

**Solutions:**
1. Ensure `execution_id` is unique and consistent
2. Check for multiple webhook calls in n8n
3. WhiteBoxXAI handles duplicates automatically, but verify n8n configuration

### Issue 4: Authentication Errors

**Symptoms:** 401 Unauthorized responses

**Solutions:**
1. Verify API key format: `Bearer wbai_...`
2. Check organization access for API key
3. Ensure API key has `webhooks:write` permission
4. Test API key with `/api/v1/webhooks/n8n/config` endpoint

### Issue 5: Node Status Not Updating

**Symptoms:** Workflows stuck in "running" status

**Solutions:**
1. Ensure `status` field is set correctly (`success` or `error`)
2. Verify `finished_at` timestamp is sent
3. Check that webhook fires after all nodes complete
4. Review n8n execution mode (`manual`, `trigger`, etc.)

---

## API Reference

### POST /api/v1/webhooks/n8n/execution

Receive n8n workflow execution data.

**Authentication:** Bearer Token

**Request Body:** N8NWorkflowExecution

**Response:** N8NWebhookResponse

**Status Codes:**
- `200` - Success
- `401` - Unauthorized
- `422` - Validation Error
- `500` - Server Error

### GET /api/v1/webhooks/n8n/config

Get webhook configuration and setup instructions.

**Authentication:** Bearer Token

**Response:** N8NWorkflowConfig

**Example:**
```json
{
  "webhook_url": "https://api.whiteboxxai.com/api/v1/webhooks/n8n/execution",
  "api_key": "wbai_...",
  "track_all_nodes": true,
  "min_execution_time_ms": 0
}
```

### POST /api/v1/webhooks/n8n/test

Test webhook with sample data.

**Authentication:** Bearer Token

**Response:**
```json
{
  "status": "test_successful",
  "workflow_id": "uuid",
  "message": "Test webhook processed successfully"
}
```

---

## Security

### Authentication

All webhook requests require:
- Valid Bearer token in `Authorization` header
- Token associated with active organization
- Proper permissions (`webhooks:write`)

### Data Privacy

- Webhook data is organization-scoped
- Only users within the organization can view workflow data
- Node input/output data is stored as JSONB (encrypted at rest)
- API keys never logged or stored in plaintext

### Signature Verification (Coming Soon)

Future versions will support webhook signature verification:

```
X-n8n-Signature: sha256=...
```

This will allow verification that webhooks originate from your n8n instance.

---

## FAQ

**Q: Can I use this with self-hosted n8n?**
A: Yes! The webhook integration works with both n8n Cloud and self-hosted instances.

**Q: How much data can I send in a webhook?**
A: Maximum payload size is 10MB. For larger data, store in external storage and send references.

**Q: Are webhooks processed synchronously?**
A: Yes, webhooks are processed immediately. Response typically within 100-500ms.

**Q: Can I track multiple workflows?**
A: Yes, each workflow execution is tracked separately using unique `execution_id`.

**Q: What happens if WhiteBoxXAI is down?**
A: n8n webhook will fail, but your workflow continues. Configure error handling to retry or log failures.

**Q: Can I filter which nodes are tracked?**
A: Yes, modify the Code node to filter nodes before sending. You can also use `min_execution_time_ms` in config.

---

## Related Documentation

- [Multi-Agent Workflow Observability](../sdk/multi-agent-monitoring.md)
- [API Reference](../sdk/api-reference.md)
- [CrewAI Integration](../sdk/multi-agent-monitoring.md#crewai-integration)
- [LangChain Integration](langchain-multi-agent.md)

---

## Support

For issues or questions:
- Documentation: https://docs.whiteboxxai.com
- Community: https://community.whiteboxxai.com
- Email: support@whiteboxxai.com
