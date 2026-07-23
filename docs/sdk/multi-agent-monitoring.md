# Multi-Agent Workflow Monitoring

Comprehensive guide to monitoring multi-agent AI workflows with WhiteBoxXAI.

## Overview

WhiteBoxXAI provides built-in observability for multi-agent AI systems including:

- **CrewAI** - Agent crews with roles and tasks
- **LangChain** - Multi-agent chains and graphs
- **AutoGen** - Conversational agent frameworks
- **n8n** - Workflow automation with AI agents
- **Custom** - Custom multi-agent architectures

Track agent executions, interactions, task completions, costs, and performance bottlenecks across your entire workflow.

## Table of Contents

- [Quick Start](#quick-start)
- [Concepts](#concepts)
- [CrewAI Integration](#crewai-integration)
- [REST API](#rest-api)
- [Analytics](#analytics)
- [Best Practices](#best-practices)
- [Examples](#examples)

---

## Quick Start

### Installation

```bash
pip install whiteboxxai
```

### CrewAI Example

```python
from whiteboxxai.integrations import monitor_crew
from crewai import Agent, Task, Crew, Process

# Define agents
researcher = Agent(
    role="Research Analyst",
    goal="Find accurate information on AI safety",
    backstory="Expert researcher with 10 years experience",
    tools=[search_tool, scrape_tool],
    llm=ChatOpenAI(model="gpt-4")
)

writer = Agent(
    role="Content Writer",
    goal="Write engaging technical content",
    backstory="Professional tech writer",
    tools=[writing_tool],
    llm=ChatOpenAI(model="gpt-4")
)

# Create tasks
research_task = Task(
    description="Research latest AI safety regulations in US and EU",
    expected_output="List of 10+ regulations with summaries",
    agent=researcher
)

writing_task = Task(
    description="Write 1500 word article on AI safety regulations",
    expected_output="Professional article in markdown format",
    agent=writer
)

# Create crew
crew = Crew(
    agents=[researcher, writer],
    tasks=[research_task, writing_task],
    process=Process.sequential
)

# Monitor execution
monitor = monitor_crew(
    crew=crew,
    workflow_name="AI Safety Article Generation",
    api_key="your_api_key",
    metadata={"project": "blog", "topic": "ai_safety"}
)

# Execute crew (automatically monitored)
result = crew.kickoff()

# Complete monitoring and get analytics
summary = monitor.complete_monitoring(
    outputs={"article": result, "word_count": len(result.split())}
)

print(f"Total cost: ${summary['analytics']['metrics']['total_cost']}")
print(f"Total tokens: {summary['analytics']['metrics']['total_tokens']}")
```

---

## Concepts

### Workflow

A **workflow** represents a complete multi-agent execution session.

**Fields:**
- `id` (UUID): Unique workflow identifier
- `name` (string): Workflow name
- `framework` (enum): crewai, langchain, autogen, n8n, custom
- `status` (enum): pending, running, completed, failed, cancelled
- `inputs` (JSON): Workflow inputs
- `outputs` (JSON): Workflow outputs
- `metadata` (JSON): Custom metadata
- `tags` (array): Searchable tags
- `total_tokens` (int): Total tokens consumed
- `total_cost` (float): Total cost in USD
- `started_at` (timestamp): Start time
- `completed_at` (timestamp): End time
- `duration_ms` (int): Duration in milliseconds

### Agent

An **agent** is an AI entity with a specific role, capabilities, and model.

**Fields:**
- `id` (UUID): Unique agent identifier
- `name` (string): Agent name
- `role` (string): Agent role (researcher, writer, analyst, etc.)
- `agent_type` (string): Framework-specific type
- `model_name` (string): LLM model (gpt-4, claude-3, etc.)
- `llm_provider` (string): Provider (openai, anthropic, etc.)
- `llm_config` (JSON): Model configuration
- `system_prompt` (text): Agent system prompt
- `goal` (text): Agent goal
- `backstory` (text): Agent backstory
- `tools` (array): Available tools
- `capabilities` (array): Agent capabilities
- `total_executions` (int): Execution count
- `total_tokens` (int): Tokens consumed
- `total_cost` (float): Cost in USD

### Execution

An **execution** represents a single agent run.

**Fields:**
- `id` (UUID): Unique execution identifier
- `agent_id` (UUID): Associated agent
- `parent_execution_id` (UUID): Parent execution (for hierarchical/delegated runs)
- `execution_order` (int): Sequential order
- `status` (enum): pending, running, completed, failed, cancelled
- `inputs` (JSON): Execution inputs
- `outputs` (JSON): Execution outputs
- `llm_calls` (int): LLM API calls made
- `tool_calls` (int): Tool invocations
- `tokens_used` (int): Tokens consumed
- `cost` (float): Execution cost
- `started_at` (timestamp): Start time
- `completed_at` (timestamp): End time
- `duration_ms` (int): Duration in milliseconds
- `span_id` (string): OpenTelemetry span ID

### Interaction

An **interaction** represents agent-to-agent communication.

**Interaction Types:**
- **delegation**: Agent delegates task to another agent
- **handoff**: Agent passes control to another agent
- **query**: Agent requests information from another agent
- **feedback**: Agent provides feedback to another agent

**Fields:**
- `id` (UUID): Unique interaction identifier
- `interaction_type` (enum): Type of interaction
- `from_agent_id` (UUID): Source agent
- `to_agent_id` (UUID): Target agent
- `from_execution_id` (UUID): Source execution (optional)
- `to_execution_id` (UUID): Target execution (optional)
- `message` (text): Interaction message
- `metadata` (JSON): Additional context
- `timestamp` (timestamp): Interaction time

### Task

A **task** represents a discrete unit of work in the workflow.

**Fields:**
- `id` (UUID): Unique task identifier
- `task_name` (string): Task name
- `description` (text): Task description
- `task_type` (enum): research, writing, analysis, review, custom
- `expected_output` (text): Expected output description
- `status` (enum): pending, in_progress, completed, failed, cancelled
- `input_data` (JSON): Task inputs
- `output_data` (JSON): Task outputs
- `context` (JSON): Additional context
- `agent_id` (UUID): Assigned agent
- `parent_task_id` (UUID): Parent task (for subtasks)
- `priority` (int): Task priority
- `retries` (int): Retry count
- `tools_used` (array): Tools invoked
- `assigned_at` (timestamp): Assignment time
- `started_at` (timestamp): Start time
- `completed_at` (timestamp): End time
- `duration_ms` (int): Duration in milliseconds

---

## CrewAI Integration

### Installation

```bash
pip install whiteboxxai crewai
```

### Basic Usage

#### Option 1: Helper Function

```python
from whiteboxxai.integrations import monitor_crew

monitor = monitor_crew(
    crew=my_crew,
    workflow_name="My Workflow",
    api_key="your_api_key",
    metadata={"project": "demo"}
)

result = my_crew.kickoff()

summary = monitor.complete_monitoring(outputs={"result": result})
```

#### Option 2: CrewAIMonitor Class

```python
from whiteboxxai.integrations import CrewAIMonitor

monitor = CrewAIMonitor(
    api_key="your_api_key",
    api_url="https://api.whiteboxxai.com"  # Optional
)

workflow_id = monitor.start_monitoring(
    crew=my_crew,
    workflow_name="My Workflow",
    metadata={"project": "demo", "version": "1.0"}
)

# Execute crew
result = my_crew.kickoff()

# Log custom agent executions (optional - auto-tracked for standard crews)
monitor.log_agent_execution(
    agent=researcher,
    inputs={"query": "AI safety"},
    outputs={"findings": [...] },
    tokens_used=1500,
    cost=0.023
)

# Log agent interactions (optional)
monitor.log_interaction(
    from_agent=researcher,
    to_agent=writer,
    interaction_type="delegation",
    message="Please write article based on these findings"
)

# Complete monitoring
summary = monitor.complete_monitoring(
    status="completed",
    outputs={"article": result}
)
```

### What Gets Tracked

#### Automatically Tracked:
- ✅ Workflow creation and lifecycle
- ✅ All agents with roles, goals, tools, models
- ✅ All tasks with assignments and expected outputs
- ✅ Sequential/hierarchical execution order
- ✅ Token usage and costs (from LLM callbacks)
- ✅ Execution timestamps and durations

#### Manually Log (Optional):
- 📝 Agent-to-agent interactions (delegations, handoffs)
- 📝 Custom execution metadata
- 📝 Task-level updates
- 📝 Error details

### Advanced Features

#### Hierarchical Executions

Track parent-child agent executions for delegated tasks:

```python
# Parent execution
parent_exec_id = monitor.create_agent_execution(
    agent=manager_agent,
    inputs={"task": "coordinate_research"}
)

# Child execution (delegated)
child_exec_id = monitor.create_agent_execution(
    agent=researcher_agent,
    inputs={"subtask": "find_sources"},
    parent_execution_id=parent_exec_id
)
```

#### Custom Task Tracking

```python
from whiteboxxai.schemas.agent_workflow import TaskCreate

task_id = monitor.create_task(
    workflow_id=workflow_id,
    task_data=TaskCreate(
        task_name="Verify Facts",
        description="Cross-reference all claims",
        task_type="review",
        expected_output="Verified fact list",
        agent_id=reviewer_agent_id,
        priority=1,
        input_data={"claims": [...] },
        context={"sources": [...] }
    )
)

# Update task status
monitor.update_task_status(
    task_id=task_id,
    status="completed",
    output_data={"verified_count": 12, "issues": 2}
)
```

---

## REST API

Complete REST API for programmatic access.

### Base URL

```
https://api.whiteboxxai.com/api/v1/workflows/multi-agent
```

### Authentication

```bash
curl -H "Authorization: Bearer YOUR_API_KEY" \
     https://api.whiteboxxai.com/api/v1/workflows/multi-agent
```

### Endpoints

#### Create Workflow

```http
POST /api/v1/workflows/multi-agent/start
```

**Request:**
```json
{
  "name": "Research & Writing Workflow",
  "framework": "crewai",
  "metadata": {
    "project": "blog-automation",
    "version": "1.0"
  },
  "tags": ["content", "research"]
}
```

**Response:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "Research & Writing Workflow",
  "framework": "crewai",
  "status": "pending",
  "created_at": "2024-01-27T10:00:00Z"
}
```

#### Start Workflow

```http
POST /api/v1/workflows/multi-agent/{workflow_id}/start
```

**Request:**
```json
{
  "inputs": {
    "topic": "AI Safety",
    "target_length": 1500
  }
}
```

#### Complete Workflow

```http
POST /api/v1/workflows/multi-agent/{workflow_id}/complete?trigger_analytics=true
```

**Request:**
```json
{
  "status": "completed",
  "outputs": {
    "article": "...",
    "word_count": 1523
  }
}
```

#### Register Agent

```http
POST /api/v1/workflows/multi-agent/{workflow_id}/agents
```

**Request:**
```json
{
  "name": "Research Analyst",
  "role": "researcher",
  "agent_type": "crewai_agent",
  "model_name": "gpt-4",
  "llm_provider": "openai",
  "goal": "Find accurate information",
  "backstory": "Expert researcher",
  "tools": ["search", "scrape"],
  "llm_config": {
    "temperature": 0.7,
    "max_tokens": 2000
  }
}
```

#### List Workflows

```http
GET /api/v1/workflows/multi-agent?status=completed&framework=crewai&skip=0&limit=50
```

**Response:**
```json
{
  "workflows": [
    {
      "id": "...",
      "name": "AI Safety Article",
      "framework": "crewai",
      "status": "completed",
      "total_tokens": 15420,
      "total_cost": 0.2313,
      "created_at": "2024-01-27T10:00:00Z"
    }
  ],
  "total": 1,
  "skip": 0,
  "limit": 50
}
```

#### Get Workflow Analytics

```http
GET /api/v1/workflows/multi-agent/{workflow_id}/analytics
```

**Response:**
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

#### Get Cost Breakdown

```http
GET /api/v1/workflows/multi-agent/{workflow_id}/cost-breakdown
```

**Response:**
```json
{
  "workflow_id": "...",
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

#### Get Bottlenecks

```http
GET /api/v1/workflows/multi-agent/{workflow_id}/bottlenecks
```

**Response:**
```json
{
  "workflow_id": "...",
  "slowest_agents": [
    {
      "agent_id": "...",
      "agent_name": "Research Analyst",
      "avg_duration_ms": 3421.5,
      "execution_count": 1
    }
  ],
  "slowest_tasks": [
    {
      "task_id": "...",
      "task_name": "Research AI Safety",
      "duration_ms": 3421,
      "status": "completed"
    }
  ],
  "failing_agents": []
}
```

#### Get Timeline

```http
GET /api/v1/workflows/multi-agent/{workflow_id}/timeline
```

**Response:**
```json
{
  "workflow_id": "...",
  "events": [
    {
      "event_type": "workflow_start",
      "timestamp": "2024-01-27T10:00:00Z",
      "description": "Workflow started"
    },
    {
      "event_type": "agent_execution",
      "timestamp": "2024-01-27T10:00:05Z",
      "agent_id": "...",
      "agent_name": "Research Analyst",
      "description": "Agent execution started"
    }
  ]
}
```

See [API Reference](#) for complete endpoint documentation.

---

## Analytics

WhiteBoxXAI provides comprehensive analytics for multi-agent workflows.

### Metrics

**Workflow-level:**
- Total tokens consumed
- Total cost (USD)
- Total executions
- Completed vs failed tasks
- Average execution duration
- Workflow duration

**Agent-level:**
- Per-agent token usage
- Per-agent cost attribution
- Execution count
- Average duration
- Failure rate

**Task-level:**
- Task completion rate
- Task duration
- Retries
- Tools used

### Cost Attribution

Understand which agents consume the most resources:

```python
breakdown = monitor.get_analytics()["cost_breakdown"]

for agent in breakdown["agents"]:
    print(f"{agent['agent_name']}: ${agent['total_cost']:.4f} ({agent['total_tokens']} tokens)")
```

**Output:**
```
Research Analyst: $0.1275 (8500 tokens)
Content Writer: $0.1038 (6920 tokens)
```

### Bottleneck Analysis

Identify performance bottlenecks:

```python
bottlenecks = monitor.get_bottlenecks()

print("Slowest Agents:")
for agent in bottlenecks["slowest_agents"]:
    print(f"  {agent['agent_name']}: {agent['avg_duration_ms']/1000:.2f}s")

print("\nSlowest Tasks:")
for task in bottlenecks["slowest_tasks"]:
    print(f"  {task['task_name']}: {task['duration_ms']/1000:.2f}s")

print("\nFailing Agents:")
for agent in bottlenecks["failing_agents"]:
    print(f"  {agent['agent_name']}: {agent['failure_rate']*100:.1f}% failure rate")
```

### Timeline Reconstruction

View chronological event timeline:

```python
timeline = monitor.get_timeline()

for event in timeline["events"]:
    print(f"[{event['timestamp']}] {event['event_type']}: {event['description']}")
```

**Output:**
```
[2024-01-27T10:00:00Z] workflow_start: Workflow started
[2024-01-27T10:00:05Z] agent_execution: Research Analyst execution started
[2024-01-27T10:03:26Z] task_complete: Research AI Safety completed
[2024-01-27T10:03:30Z] interaction: Research Analyst delegated to Content Writer
[2024-01-27T10:03:35Z] agent_execution: Content Writer execution started
[2024-01-27T10:06:25Z] task_complete: Write article completed
[2024-01-27T10:06:30Z] workflow_complete: Workflow completed
```

---

## Best Practices

### 1. Always Complete Monitoring

Ensure `complete_monitoring()` is called even on errors:

```python
monitor = monitor_crew(crew, "My Workflow", api_key)

try:
    result = crew.kickoff()
    monitor.complete_monitoring(
        status="completed",
        outputs={"result": result}
    )
except Exception as e:
    monitor.complete_monitoring(
        status="failed",
        error_message=str(e)
    )
    raise
```

### 2. Use Meaningful Metadata

Add searchable metadata for filtering:

```python
monitor = monitor_crew(
    crew=crew,
    workflow_name="Article Generation",
    api_key=api_key,
    metadata={
        "project": "blog",
        "topic": "ai_safety",
        "author": "john_doe",
        "version": "2.1",
        "environment": "production"
    }
)
```

### 3. Tag Workflows

Use tags for categorization:

```python
workflow_data = WorkflowCreate(
    name="Research Workflow",
    framework="crewai",
    tags=["research", "automated", "high-priority"]
)
```

### 4. Monitor Critical Interactions

Log important agent interactions:

```python
# When one agent delegates to another
monitor.log_interaction(
    from_agent=manager,
    to_agent=specialist,
    interaction_type="delegation",
    message=f"Delegating specialized task: {task_description}"
)

# When agent requests feedback
monitor.log_interaction(
    from_agent=writer,
    to_agent=reviewer,
    interaction_type="query",
    message="Please review draft for accuracy"
)
```

### 5. Track Hierarchical Executions

Use `parent_execution_id` for delegated tasks:

```python
execution_data = ExecutionCreate(
    agent_id=subordinate_agent_id,
    inputs={"subtask": "..."},
    parent_execution_id=manager_execution_id,
    execution_order=2
)
```

### 6. Set Appropriate Task Priorities

```python
task_data = TaskCreate(
    task_name="Critical Security Review",
    priority=1,  # Higher priority
    agent_id=security_agent_id
)
```

### 7. Monitor Costs Regularly

```python
# Get cost breakdown after completion
breakdown = monitor.get_analytics()["cost_breakdown"]

# Alert if cost exceeds threshold
if breakdown["total_cost"] > 1.0:
    send_alert(f"Workflow cost ${breakdown['total_cost']:.2f} exceeds $1.00 threshold")
```

---

## Examples

### Example 1: Multi-Stage Research Pipeline

```python
from whiteboxxai.integrations import CrewAIMonitor
from crewai import Agent, Task, Crew, Process

# Initialize monitor
monitor = CrewAIMonitor(api_key="your_api_key")

# Create agents
researcher = Agent(
    role="Research Analyst",
    goal="Find comprehensive information",
    tools=[search_tool, scrape_tool],
    llm=ChatOpenAI(model="gpt-4")
)

fact_checker = Agent(
    role="Fact Checker",
    goal="Verify accuracy of information",
    tools=[fact_check_tool],
    llm=ChatOpenAI(model="gpt-4")
)

summarizer = Agent(
    role="Summarizer",
    goal="Create concise summaries",
    llm=ChatOpenAI(model="gpt-3.5-turbo")
)

# Create tasks
research_task = Task(
    description="Research AI safety regulations in US, EU, UK",
    agent=researcher
)

fact_check_task = Task(
    description="Verify all claims and sources",
    agent=fact_checker
)

summary_task = Task(
    description="Summarize findings in 500 words",
    agent=summarizer
)

# Create crew
crew = Crew(
    agents=[researcher, fact_checker, summarizer],
    tasks=[research_task, fact_check_task, summary_task],
    process=Process.sequential
)

# Monitor execution
workflow_id = monitor.start_monitoring(
    crew=crew,
    workflow_name="AI Safety Research Pipeline",
    metadata={"project": "regulatory_compliance", "deadline": "2024-02-01"}
)

try:
    result = crew.kickoff()

    summary = monitor.complete_monitoring(
        status="completed",
        outputs={"summary": result}
    )

    # Print analytics
    metrics = summary["analytics"]["metrics"]
    print(f"✅ Research pipeline completed")
    print(f"   Total cost: ${metrics['total_cost']:.4f}")
    print(f"   Total tokens: {metrics['total_tokens']}")
    print(f"   Duration: {metrics.get('duration_ms', 0)/1000:.1f}s")

except Exception as e:
    monitor.complete_monitoring(
        status="failed",
        error_message=str(e)
    )
    print(f"❌ Research pipeline failed: {str(e)}")
```

### Example 2: Content Creation with Feedback Loop

```python
from whiteboxxai.integrations import monitor_crew

# Create agents with feedback capability
writer = Agent(
    role="Content Writer",
    goal="Write engaging articles",
    allow_delegation=True,  # Can request feedback
    llm=ChatOpenAI(model="gpt-4")
)

editor = Agent(
    role="Editor",
    goal="Improve content quality",
    backstory="Experienced editor with high standards",
    llm=ChatOpenAI(model="gpt-4")
)

# Tasks with iterative feedback
draft_task = Task(
    description="Write initial draft about quantum computing",
    expected_output="1000 word article draft",
    agent=writer
)

edit_task = Task(
    description="Review and provide feedback on draft",
    expected_output="Edited article with suggestions",
    agent=editor
)

final_task = Task(
    description="Incorporate feedback and finalize",
    expected_output="Polished final article",
    agent=writer
)

crew = Crew(
    agents=[writer, editor],
    tasks=[draft_task, edit_task, final_task],
    process=Process.sequential
)

# Monitor with interaction tracking
monitor = monitor_crew(
    crew=crew,
    workflow_name="Content Creation with Editing",
    api_key="your_api_key",
    metadata={"content_type": "technical_article"}
)

# Log feedback interaction
monitor.log_interaction(
    from_agent=writer,
    to_agent=editor,
    interaction_type="query",
    message="Please review draft for technical accuracy and clarity"
)

result = crew.kickoff()

monitor.log_interaction(
    from_agent=editor,
    to_agent=writer,
    interaction_type="feedback",
    message="Suggested improvements: clarify quantum entanglement section, add examples"
)

summary = monitor.complete_monitoring(
    outputs={"article": result, "word_count": len(result.split())}
)

# Analyze interactions
interactions = monitor.get_workflow_interactions()
print(f"Total interactions: {len(interactions)}")
```

### Example 3: Cost-Optimized Workflow

```python
from whiteboxxai.integrations import CrewAIMonitor

monitor = CrewAIMonitor(api_key="your_api_key")

# Mix of expensive and cheap agents
expensive_agent = Agent(
    role="Expert Analyst",
    llm=ChatOpenAI(model="gpt-4"),  # $0.03/1k tokens
    goal="Deep analysis"
)

cheap_agent = Agent(
    role="Formatter",
    llm=ChatOpenAI(model="gpt-3.5-turbo"),  # $0.001/1k tokens
    goal="Format output"
)

crew = Crew(
    agents=[expensive_agent, cheap_agent],
    tasks=[analysis_task, formatting_task]
)

workflow_id = monitor.start_monitoring(crew, "Cost-Optimized Workflow")

result = crew.kickoff()

summary = monitor.complete_monitoring(outputs={"result": result})

# Analyze cost breakdown
breakdown = summary["analytics"]["cost_breakdown"]

for agent in breakdown["agents"]:
    cost_per_exec = agent["total_cost"] / agent["execution_count"]
    print(f"{agent['agent_name']}:")
    print(f"  Cost: ${agent['total_cost']:.4f}")
    print(f"  Tokens: {agent['total_tokens']}")
    print(f"  Cost/execution: ${cost_per_exec:.4f}")
```

---

## Troubleshooting

### Issue: Workflow not appearing in dashboard

**Solution:** Ensure `complete_monitoring()` was called:

```python
try:
    result = crew.kickoff()
finally:
    # Always complete even on error
    monitor.complete_monitoring(
        status="completed" if result else "failed"
    )
```

### Issue: Missing agent executions

**Solution:** Agents must be registered before execution:

```python
# Agents are auto-registered from crew.agents
# For manual registration:
agent_id = monitor._register_agent(custom_agent)
```

### Issue: Incorrect cost attribution

**Solution:** Ensure LLM callbacks are configured:

```python
from langchain.callbacks import WhiteBoxXAICallbackHandler

llm = ChatOpenAI(
    model="gpt-4",
    callbacks=[WhiteBoxXAICallbackHandler()]  # Auto-tracks tokens/cost
)
```

### Issue: Analytics not available

**Solution:** Analytics are calculated asynchronously after workflow completion. Wait 10-30 seconds then query:

```python
import time

monitor.complete_monitoring(status="completed")

# Wait for async analytics
time.sleep(15)

analytics = monitor.get_analytics()
```

---

## Next Steps

- [LangChain Multi-Agent Monitoring](../integrations/langchain-multi-agent.md)
- [n8n Webhook Integration](../integrations/n8n-webhooks.md)
- [Multi-Agent Quick Reference](multi-agent-quick-reference.md)
- [API Reference](api-reference.md)

---

## Support

- Documentation: https://docs.whiteboxxai.com
- API Reference: https://api.whiteboxxai.com/docs
- GitHub Issues: https://github.com/whiteboxxai/whiteboxxai/issues
- Email: support@whiteboxxai.com
