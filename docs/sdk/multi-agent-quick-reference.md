# Multi-Agent Workflow Monitoring - Quick Reference

Fast reference for monitoring multi-agent AI workflows with WhiteBoxXAI.

## Installation

```bash
pip install whitebox-xai-sdk
```

## CrewAI - Quick Start

```python
from whiteboxxai.integrations import monitor_crew
from crewai import Agent, Task, Crew

# Monitor crew
monitor = monitor_crew(
    crew=my_crew,
    workflow_name="My Workflow",
    api_key="your_api_key"
)

# Execute
result = my_crew.kickoff()

# Complete
summary = monitor.complete_monitoring(outputs={"result": result})
print(f"Cost: ${summary['analytics']['metrics']['total_cost']:.4f}")
```

## API Endpoints

### Workflows

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/v1/workflows/multi-agent/start` | Create workflow |
| `POST` | `/api/v1/workflows/multi-agent/{id}/start` | Start workflow |
| `POST` | `/api/v1/workflows/multi-agent/{id}/complete` | Complete workflow |
| `GET` | `/api/v1/workflows/multi-agent` | List workflows |
| `GET` | `/api/v1/workflows/multi-agent/{id}` | Get workflow |
| `DELETE` | `/api/v1/workflows/multi-agent/{id}` | Delete workflow |

### Agents

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/v1/workflows/multi-agent/{id}/agents` | Register agent |
| `GET` | `/api/v1/workflows/multi-agent/{id}/agents` | List agents |

### Executions

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/v1/workflows/multi-agent/{id}/executions` | Create execution |

### Interactions

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/v1/workflows/multi-agent/{id}/interactions` | Log interaction |
| `GET` | `/api/v1/workflows/multi-agent/{id}/interactions` | List interactions |

### Tasks

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/v1/workflows/multi-agent/{id}/tasks` | Create task |
| `PATCH` | `/api/v1/workflows/multi-agent/tasks/{id}` | Update task |
| `GET` | `/api/v1/workflows/multi-agent/{id}/tasks` | List tasks |

### Analytics

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v1/workflows/multi-agent/{id}/analytics` | Get metrics |
| `GET` | `/api/v1/workflows/multi-agent/{id}/cost-breakdown` | Get cost attribution |
| `GET` | `/api/v1/workflows/multi-agent/{id}/bottlenecks` | Get bottlenecks |
| `GET` | `/api/v1/workflows/multi-agent/{id}/timeline` | Get timeline |

## SDK Methods

### CrewAIMonitor

```python
from whiteboxxai.integrations import CrewAIMonitor

monitor = CrewAIMonitor(api_key="key", api_url="https://api.whiteboxxai.com")
```

#### Core Methods

| Method | Parameters | Returns | Description |
|--------|------------|---------|-------------|
| `start_monitoring()` | `crew, workflow_name, metadata` | `workflow_id` | Start monitoring |
| `complete_monitoring()` | `status, outputs, error_message` | `summary` | Complete monitoring |
| `log_agent_execution()` | `agent, inputs, outputs, tokens, cost` | `None` | Log execution |
| `log_interaction()` | `from_agent, to_agent, type, message` | `None` | Log interaction |
| `log_task_completion()` | `task, status, output_data` | `None` | Log task |
| `get_analytics()` | - | `dict` | Get analytics |

#### Helper Function

```python
from whiteboxxai.integrations import monitor_crew

monitor = monitor_crew(
    crew=my_crew,
    workflow_name="Workflow",
    api_key="key",
    metadata={"key": "value"}
)
```

## Request Examples

### Create Workflow

```bash
curl -X POST https://api.whiteboxxai.com/api/v1/workflows/multi-agent/start \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Research Workflow",
    "framework": "crewai",
    "metadata": {"project": "demo"},
    "tags": ["research"]
  }'
```

### Register Agent

```bash
curl -X POST https://api.whiteboxxai.com/api/v1/workflows/multi-agent/{workflow_id}/agents \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Researcher",
    "role": "researcher",
    "agent_type": "crewai_agent",
    "model_name": "gpt-4",
    "llm_provider": "openai",
    "goal": "Find information",
    "tools": ["search"],
    "llm_config": {"temperature": 0.7}
  }'
```

### Complete Workflow

```bash
curl -X POST https://api.whiteboxxai.com/api/v1/workflows/multi-agent/{workflow_id}/complete \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "status": "completed",
    "outputs": {"result": "..."}
  }'
```

### Get Analytics

```bash
curl https://api.whiteboxxai.com/api/v1/workflows/multi-agent/{workflow_id}/analytics \
  -H "Authorization: Bearer YOUR_API_KEY"
```

## Response Examples

### Workflow Response

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "Research Workflow",
  "framework": "crewai",
  "status": "completed",
  "total_tokens": 15420,
  "total_cost": 0.2313,
  "started_at": "2024-01-27T10:00:00Z",
  "completed_at": "2024-01-27T10:06:30Z",
  "duration_ms": 390000
}
```

### Analytics Response

```json
{
  "total_tokens": 15420,
  "total_cost": 0.2313,
  "total_executions": 3,
  "completed_tasks": 5,
  "failed_tasks": 0,
  "average_execution_duration_ms": 2345.67
}
```

### Cost Breakdown Response

```json
{
  "workflow_id": "550e8400-e29b-41d4-a716-446655440000",
  "total_cost": 0.2313,
  "agents": [
    {
      "agent_id": "...",
      "agent_name": "Research Analyst",
      "agent_role": "researcher",
      "total_tokens": 8500,
      "total_cost": 0.1275,
      "execution_count": 1,
      "avg_duration_ms": 3421.5
    }
  ]
}
```

## Enums

### Workflow Status

- `pending` - Created, not started
- `running` - Currently executing
- `completed` - Successfully completed
- `failed` - Failed with error
- `cancelled` - Manually cancelled

### Framework

- `crewai` - CrewAI framework
- `langchain` - LangChain multi-agent
- `autogen` - Microsoft AutoGen
- `n8n` - n8n workflow automation
- `custom` - Custom implementation

### Interaction Type

- `delegation` - Task delegated to another agent
- `handoff` - Control passed to another agent
- `query` - Information requested from another agent
- `feedback` - Feedback provided to another agent

### Task Status

- `pending` - Not started
- `in_progress` - Currently executing
- `completed` - Successfully completed
- `failed` - Failed with error
- `cancelled` - Manually cancelled

### Task Type

- `research` - Research/information gathering
- `writing` - Content creation
- `analysis` - Data/information analysis
- `review` - Review/validation
- `custom` - Custom task type

## Code Snippets

### Error Handling

```python
monitor = monitor_crew(crew, "Workflow", api_key)

try:
    result = crew.kickoff()
    monitor.complete_monitoring(status="completed", outputs={"result": result})
except Exception as e:
    monitor.complete_monitoring(status="failed", error_message=str(e))
    raise
```

### Cost Tracking

```python
summary = monitor.complete_monitoring(outputs={"result": result})
cost = summary["analytics"]["metrics"]["total_cost"]

if cost > 1.0:
    print(f"⚠️  High cost: ${cost:.4f}")
```

### Hierarchical Execution

```python
parent_exec = monitor.create_agent_execution(
    agent=manager,
    inputs={"task": "coordinate"}
)

child_exec = monitor.create_agent_execution(
    agent=worker,
    inputs={"subtask": "..."},
    parent_execution_id=parent_exec
)
```

### Custom Metadata

```python
monitor = monitor_crew(
    crew=crew,
    workflow_name="Workflow",
    api_key=api_key,
    metadata={
        "project": "blog",
        "author": "john",
        "environment": "prod",
        "version": "2.1"
    }
)
```

### Interaction Logging

```python
# Delegation
monitor.log_interaction(
    from_agent=manager,
    to_agent=specialist,
    interaction_type="delegation",
    message="Please handle specialized task"
)

# Feedback
monitor.log_interaction(
    from_agent=reviewer,
    to_agent=writer,
    interaction_type="feedback",
    message="Excellent work, minor formatting fixes needed"
)
```

### Async Analytics

```python
import time

# Complete workflow (triggers async analytics)
monitor.complete_monitoring(status="completed")

# Wait for analytics to process
time.sleep(15)

# Get analytics
analytics = monitor.get_analytics()
breakdown = monitor.get_cost_breakdown()
bottlenecks = monitor.get_bottlenecks()
timeline = monitor.get_timeline()
```

## Database Schema

### agent_workflows

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `organization_id` | UUID | Organization FK |
| `user_id` | UUID | User FK |
| `name` | String | Workflow name |
| `framework` | Enum | Framework type |
| `status` | Enum | Workflow status |
| `inputs` | JSONB | Input data |
| `outputs` | JSONB | Output data |
| `total_tokens` | Integer | Total tokens |
| `total_cost` | Float | Total cost USD |
| `trace_id` | String(32) | OpenTelemetry trace ID |
| `metadata` | JSONB | Custom metadata |
| `tags` | Array | Tags |

### workflow_agents

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `workflow_id` | UUID | Workflow FK |
| `name` | String | Agent name |
| `role` | String | Agent role |
| `agent_type` | String | Type |
| `model_name` | String | LLM model |
| `llm_provider` | String | LLM provider |
| `goal` | Text | Agent goal |
| `tools` | Array | Tools |
| `total_executions` | Integer | Execution count |
| `total_tokens` | Integer | Tokens used |
| `total_cost` | Float | Cost USD |

### agent_executions

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `workflow_id` | UUID | Workflow FK |
| `agent_id` | UUID | Agent FK |
| `parent_execution_id` | UUID | Parent FK (nullable) |
| `execution_order` | Integer | Sequential order |
| `status` | Enum | Status |
| `inputs` | JSONB | Inputs |
| `outputs` | JSONB | Outputs |
| `tokens_used` | Integer | Tokens |
| `cost` | Float | Cost USD |
| `span_id` | String(16) | OTEL span ID |

### agent_interactions

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `workflow_id` | UUID | Workflow FK |
| `interaction_type` | Enum | Type |
| `from_agent_id` | UUID | Source agent |
| `to_agent_id` | UUID | Target agent |
| `message` | Text | Message |
| `metadata` | JSONB | Custom data |
| `timestamp` | Timestamp | Time |

### agent_tasks

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `workflow_id` | UUID | Workflow FK |
| `task_name` | String | Task name |
| `task_type` | Enum | Type |
| `status` | Enum | Status |
| `agent_id` | UUID | Assigned agent FK |
| `parent_task_id` | UUID | Parent task FK |
| `input_data` | JSONB | Inputs |
| `output_data` | JSONB | Outputs |
| `priority` | Integer | Priority |
| `duration_ms` | Integer | Duration |

## Common Patterns

### Simple Sequential Workflow

```python
monitor = monitor_crew(crew, "Simple Workflow", api_key)
result = crew.kickoff()
monitor.complete_monitoring(outputs={"result": result})
```

### Production with Error Handling

```python
monitor = None
try:
    monitor = monitor_crew(crew, "Prod Workflow", api_key,
                          metadata={"env": "prod"})
    result = crew.kickoff()
    monitor.complete_monitoring(status="completed", outputs={"result": result})
except Exception as e:
    if monitor:
        monitor.complete_monitoring(status="failed", error_message=str(e))
    logger.error(f"Workflow failed: {e}")
    raise
```

### Cost-Aware Execution

```python
monitor = monitor_crew(crew, "Cost-Aware", api_key)
result = crew.kickoff()
summary = monitor.complete_monitoring(outputs={"result": result})

cost = summary["analytics"]["metrics"]["total_cost"]
if cost > COST_THRESHOLD:
    send_alert(f"High cost: ${cost:.4f}")
```

### Analytics After Completion

```python
monitor = monitor_crew(crew, "Analytics Workflow", api_key)
result = crew.kickoff()
monitor.complete_monitoring(outputs={"result": result})

time.sleep(15)  # Wait for async analytics

analytics = monitor.get_analytics()
breakdown = analytics["cost_breakdown"]
metrics = analytics["metrics"]

print(f"Total: ${metrics['total_cost']:.4f}")
for agent in breakdown["agents"]:
    print(f"  {agent['agent_name']}: ${agent['total_cost']:.4f}")
```

---

## Environment Variables

```bash
export WhiteBoxXAI_API_KEY="your_api_key"
export WhiteBoxXAI_API_URL="https://api.whiteboxxai.com"  # Optional
```

## Links

- [Full Documentation](multi-agent-monitoring.md)
- [API Reference](api-reference.md)
- [Dashboard](https://app.whiteboxxai.com)
- [Python SDK on GitHub](https://github.com/AgentaFlow/whitebox-python-sdk)
