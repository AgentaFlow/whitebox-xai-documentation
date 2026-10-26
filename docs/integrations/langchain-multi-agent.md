# LangChain Multi-Agent Integration Guide

Complete guide to monitoring LangChain multi-agent workflows with WhiteBoxXAI.

## Table of Contents

- [Overview](#overview)
- [Quick Start](#quick-start)
- [Components](#components)
- [Usage Examples](#usage-examples)
- [Best Practices](#best-practices)
- [API Reference](#api-reference)

## Overview

The WhiteBoxXAI LangChain integration provides comprehensive monitoring for LangChain-based multi-agent systems, including:

- **MultiAgentCallbackHandler**: Automatic tracking of agent executions, tool calls, and LLM usage
- **LangGraphMultiAgentMonitor**: High-level monitoring for LangGraph workflows with multiple agents
- **Helper Functions**: Simple one-line monitoring for common patterns

### Supported Patterns

✅ Single agent workflows (ReAct, Plan-and-Execute)
✅ LangGraph multi-agent patterns
✅ Agent supervisors and coordinators
✅ Sequential and parallel agent execution
✅ Agent-to-agent communication and handoffs
✅ Tool usage tracking across agents
✅ Token and cost attribution per agent

## Quick Start

### Installation

```bash
pip install whitebox-xai-sdk langchain
```

### Basic Single Agent Monitoring

```python
from langchain.agents import AgentExecutor, create_react_agent
from whiteboxxai import WhiteBoxXAI
from whiteboxxai.integrations import monitor_langchain_agent

# Initialize client
client = WhiteBoxXAI(api_key="your_api_key")

# Create your agent
agent_executor = AgentExecutor(agent=agent, tools=tools)

# Monitor execution (one function call!)
result_dict = monitor_langchain_agent(
    client=client,
    agent_executor=agent_executor,
    workflow_name="Research Task",
    agent_name="researcher",
    inputs={"input": "Research AI safety"}
)

print(f"Result: {result_dict['result']}")
print(f"Workflow ID: {result_dict['workflow_id']}")
print(f"Status: {result_dict['status']}")
```

### Multi-Agent LangGraph Monitoring

```python
from langgraph.graph import StateGraph
from whiteboxxai.integrations import LangGraphMultiAgentMonitor

# Create monitor
monitor = LangGraphMultiAgentMonitor(
    client=client,
    workflow_name="Multi-Agent Research Pipeline"
)

# Start monitoring
workflow_id = monitor.start_monitoring(
    inputs={"topic": "AI alignment"}
)

# Register all agents
monitor.register_agent(
    agent_name="supervisor",
    role="Coordinates research workflow",
    model_name="gpt-4"
)

monitor.register_agent(
    agent_name="researcher",
    role="Gathers information from web",
    model_name="gpt-3.5-turbo",
    tools=["search", "scrape"]
)

monitor.register_agent(
    agent_name="writer",
    role="Writes comprehensive reports",
    model_name="gpt-4",
    tools=["grammar_check"]
)

# Execute workflow with callbacks
graph = create_workflow_graph()
result = graph.invoke(
    inputs,
    config={"callbacks": monitor.get_callbacks("supervisor")}
)

# Log agent handoffs
monitor.log_handoff(
    from_agent="supervisor",
    to_agent="researcher",
    message="Please research the latest findings on AI alignment"
)

# Complete monitoring
summary = monitor.complete_monitoring(outputs={"report": result})

# View analytics
print(f"Total cost: ${summary['analytics']['total_cost']:.4f}")
print(f"Total tokens: {summary['analytics']['total_tokens']}")
print(f"Duration: {summary['analytics']['duration_seconds']}s")
```

## Components

### 1. MultiAgentCallbackHandler

The core callback handler that integrates with LangChain's callback system to track all agent activities.

**Tracks:**
- Chain start/end/error events
- LLM calls and token usage
- Tool calls and results
- Agent actions and decisions
- Execution timing and status

**Example:**

```python
from whiteboxxai.integrations import MultiAgentCallbackHandler

# Create workflow first
workflow_response = client.agent_workflows.create(
    name="Research Workflow",
    framework="langchain"
)
workflow_id = workflow_response["id"]

# Start workflow
client.agent_workflows.start(workflow_id)

# Create callback for specific agent
callback = MultiAgentCallbackHandler(
    client=client,
    workflow_id=workflow_id,
    agent_name="researcher",
    agent_role="Research Agent",
    track_tokens=True,
    track_costs=True
)

# Use with agent executor
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    callbacks=[callback],
    verbose=True
)

result = agent_executor.run("Research quantum computing")

# Complete workflow
client.agent_workflows.complete(
    workflow_id,
    outputs={"result": result}
)
```

### 2. LangGraphMultiAgentMonitor

High-level monitor for LangGraph workflows with multiple cooperating agents.

**Features:**
- Automatic workflow lifecycle management
- Agent registration and tracking
- Per-agent callback management
- Handoff logging
- Analytics retrieval

**Example:**

```python
from whiteboxxai.integrations import LangGraphMultiAgentMonitor

monitor = LangGraphMultiAgentMonitor(
    client=client,
    workflow_name="Content Generation Pipeline",
    meta_data={"project": "blog-automation", "version": "2.0"}
)

# Start monitoring
workflow_id = monitor.start_monitoring(
    inputs={"topic": "Machine Learning"}
)

# Register agents
monitor.register_agent("planner", role="Plans content strategy")
monitor.register_agent("researcher", role="Gathers facts")
monitor.register_agent("writer", role="Writes articles")
monitor.register_agent("editor", role="Edits and improves")

# Build LangGraph workflow
def create_graph():
    workflow = StateGraph(AgentState)

    # Add nodes
    workflow.add_node("plan", planner_agent)
    workflow.add_node("research", researcher_agent)
    workflow.add_node("write", writer_agent)
    workflow.add_node("edit", editor_agent)

    # Add edges
    workflow.add_edge("plan", "research")
    workflow.add_edge("research", "write")
    workflow.add_edge("write", "edit")

    return workflow.compile()

graph = create_graph()

# Execute with callbacks per agent
config = {
    "callbacks": monitor.get_callbacks("planner")
}

result = graph.invoke({"topic": "Machine Learning"}, config=config)

# Log handoffs between agents
monitor.log_handoff(
    from_agent="planner",
    to_agent="researcher",
    message="Research these key topics: transformers, diffusion models",
    meta_data={"topics": ["transformers", "diffusion"]}
)

# Complete and get analytics
summary = monitor.complete_monitoring(outputs=result)
```

### 3. Helper Function: monitor_langchain_agent

Simple wrapper for monitoring single-agent executions.

**Example:**

```python
from whiteboxxai.integrations import monitor_langchain_agent

# One function call to monitor entire execution
result_dict = monitor_langchain_agent(
    client=client,
    agent_executor=my_agent,
    workflow_name="Data Analysis Task",
    agent_name="data_analyst",
    inputs={"input": "Analyze sales data for Q4"},
    input="Analyze sales data for Q4"  # Additional run kwargs
)

# Access results
if result_dict["status"] == "completed":
    print(f"Success: {result_dict['result']}")
    print(f"Workflow ID: {result_dict['workflow_id']}")
else:
    print(f"Failed: {result_dict['error']}")
```

## Usage Examples

### Example 1: ReAct Agent with Tools

```python
from langchain.agents import create_react_agent, AgentExecutor
from langchain.tools import Tool
from langchain_openai import ChatOpenAI
from whiteboxxai.integrations import MultiAgentCallbackHandler

# Define tools
tools = [
    Tool(
        name="Search",
        func=search_function,
        description="Search the web for information"
    ),
    Tool(
        name="Calculator",
        func=calculator_function,
        description="Perform mathematical calculations"
    )
]

# Create agent
llm = ChatOpenAI(model="gpt-3.5-turbo")
agent = create_react_agent(llm=llm, tools=tools, prompt=prompt_template)

# Create workflow
workflow = client.agent_workflows.create(
    name="Research & Calculate",
    framework="langchain",
    inputs={"query": "What is the square root of the year AI was invented?"}
)

client.agent_workflows.start(workflow["id"])

# Register agent
client.agent_workflows.register_agent(
    workflow_id=workflow["id"],
    name="calculator_agent",
    role="Searches and calculates",
    model_name="gpt-3.5-turbo",
    tools=["Search", "Calculator"]
)

# Create callback
callback = MultiAgentCallbackHandler(
    client=client,
    workflow_id=workflow["id"],
    agent_name="calculator_agent"
)

# Execute
executor = AgentExecutor(agent=agent, tools=tools, callbacks=[callback])
result = executor.run("What is the square root of the year AI was invented?")

# Complete
client.agent_workflows.complete(
    workflow["id"],
    outputs={"answer": result}
)

# Get analytics
analytics = client.agent_workflows.get_analytics(workflow["id"])
print(f"Total cost: ${analytics['total_cost']}")
print(f"Tool calls: {len(analytics['tool_calls'])}")
```

### Example 2: Multi-Agent LangGraph Supervisor Pattern

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict, Annotated, Sequence
from langchain.schema import HumanMessage
from whiteboxxai.integrations import LangGraphMultiAgentMonitor

# Define state
class AgentState(TypedDict):
    messages: Annotated[Sequence[HumanMessage], "conversation history"]
    next_agent: str

# Create monitor
monitor = LangGraphMultiAgentMonitor(
    client=client,
    workflow_name="Supervisor Pattern Example",
    meta_data={"pattern": "supervisor", "num_agents": 3}
)

workflow_id = monitor.start_monitoring(
    inputs={"task": "Create a blog post about AI"}
)

# Register agents
monitor.register_agent(
    "supervisor",
    role="Decides which agent should act next",
    model_name="gpt-4"
)

monitor.register_agent(
    "researcher",
    role="Researches topics",
    model_name="gpt-3.5-turbo",
    tools=["web_search", "wikipedia"]
)

monitor.register_agent(
    "writer",
    role="Writes content",
    model_name="gpt-4",
    tools=["grammar_check"]
)

# Define agent functions
def supervisor_agent(state):
    """Decides which agent should run next."""
    callbacks = monitor.get_callbacks("supervisor")
    # Your supervisor logic here
    return {"next_agent": "researcher"}

def researcher_agent(state):
    """Researches the topic."""
    callbacks = monitor.get_callbacks("researcher")
    # Your research logic here
    monitor.log_handoff(
        from_agent="researcher",
        to_agent="writer",
        message="Research complete, handing off to writer",
        meta_data={"findings_count": 5}
    )
    return {"next_agent": "writer"}

def writer_agent(state):
    """Writes the content."""
    callbacks = monitor.get_callbacks("writer")
    # Your writing logic here
    return {"next_agent": END}

# Build graph
workflow = StateGraph(AgentState)
workflow.add_node("supervisor", supervisor_agent)
workflow.add_node("researcher", researcher_agent)
workflow.add_node("writer", writer_agent)

workflow.set_entry_point("supervisor")
workflow.add_conditional_edges("supervisor", lambda x: x["next_agent"])
workflow.add_edge("researcher", "supervisor")
workflow.add_edge("writer", END)

graph = workflow.compile()

# Execute
result = graph.invoke({
    "messages": [HumanMessage(content="Create a blog post about AI")],
    "next_agent": "supervisor"
})

# Complete monitoring
summary = monitor.complete_monitoring(outputs=result)

# View results
print(f"Workflow ID: {summary['workflow_id']}")
print(f"Total cost: ${summary['analytics']['total_cost']:.4f}")
print(f"Tokens: {summary['analytics']['total_tokens']}")

# Get detailed cost breakdown
cost_breakdown = client.agent_workflows.get_cost_breakdown(workflow_id)
for agent_cost in cost_breakdown["agents"]:
    print(f"{agent_cost['agent_name']}: ${agent_cost['total_cost']:.4f}")
```

### Example 3: Error Handling and Retries

```python
from whiteboxxai.integrations import monitor_langchain_agent

try:
    result_dict = monitor_langchain_agent(
        client=client,
        agent_executor=agent_executor,
        workflow_name="Error Handling Example",
        agent_name="resilient_agent",
        inputs={"input": "Process this data"},
        max_iterations=5,
        handle_parsing_errors=True
    )

    if result_dict["status"] == "completed":
        print(f"Success: {result_dict['result']}")

        # Get timeline to see execution flow
        timeline = client.agent_workflows.get_timeline(result_dict["workflow_id"])
        for event in timeline["events"]:
            print(f"{event['timestamp']}: {event['event_type']} - {event['description']}")
    else:
        print(f"Workflow failed: {result_dict['error']}")

        # Get bottlenecks to diagnose issues
        bottlenecks = client.agent_workflows.get_bottlenecks(result_dict["workflow_id"])
        print(f"Slowest operations: {bottlenecks['slowest_agents']}")
        print(f"Failed operations: {bottlenecks['failing_agents']}")

except Exception as e:
    print(f"Unexpected error: {e}")
```

## Best Practices

### 1. Workflow Lifecycle Management

```python
# Always complete workflows
try:
    result = agent.run(input_data)
    client.agent_workflows.complete(workflow_id, outputs={"result": result})
except Exception as e:
    # Mark as failed on error
    client.agent_workflows.complete(
        workflow_id,
        outputs={"error": str(e)},
        status="failed"
    )
```

### 2. Agent Registration

Register agents before execution for better tracking:

```python
# Good: Register agents with full details
monitor.register_agent(
    agent_name="researcher",
    role="Researches academic papers and web sources",
    model_name="gpt-4",
    tools=["arxiv_search", "google_search", "wikipedia"],
    meta_data={"expertise": "scientific research"}
)

# Better: Include LLM configuration
client.agent_workflows.register_agent(
    workflow_id=workflow_id,
    name="researcher",
    role="Researches academic papers",
    model_name="gpt-4",
    llm_provider="openai",
    llm_config={
        "temperature": 0.1,
        "max_tokens": 2000,
        "top_p": 0.9
    },
    tools=["arxiv_search", "google_search"]
)
```

### 3. Handoff Logging

Log all agent-to-agent handoffs for visibility:

```python
# Log handoff with context
monitor.log_handoff(
    from_agent="supervisor",
    to_agent="researcher",
    message="Please research the following topics: X, Y, Z",
    meta_data={
        "topics": ["topic_X", "topic_Y", "topic_Z"],
        "priority": "high",
        "deadline": "2026-02-01"
    }
)
```

### 4. Cost Optimization

Track costs per agent to optimize:

```python
# After workflow completion
cost_breakdown = client.agent_workflows.get_cost_breakdown(workflow_id)

# Identify expensive agents
expensive_agents = [
    agent for agent in cost_breakdown["agents"]
    if agent["total_cost"] > 0.10
]

for agent in expensive_agents:
    print(f"{agent['agent_name']}: ${agent['total_cost']:.4f}")
    print(f"  Avg cost per execution: ${agent['avg_cost_per_execution']:.4f}")
    print(f"  Total executions: {agent['execution_count']}")
```

### 5. Metadata and Tags

Use metadata for filtering and organization:

```python
monitor = LangGraphMultiAgentMonitor(
    client=client,
    workflow_name="Content Pipeline",
    meta_data={
        "project": "blog-automation",
        "environment": "production",
        "version": "2.1.0",
        "author": "content-team",
        "tags": ["content", "ai-generated", "automated"]
    }
)
```

## API Reference

### MultiAgentCallbackHandler

```python
class MultiAgentCallbackHandler(BaseCallbackHandler):
    def __init__(
        self,
        client: WhiteBoxXAI,
        workflow_id: str,
        agent_name: str = "main",
        agent_role: Optional[str] = None,
        track_tokens: bool = True,
        track_costs: bool = True,
    )
```

**Parameters:**
- `client`: WhiteBoxXAI client instance
- `workflow_id`: ID of the workflow to track
- `agent_name`: Name of the current agent
- `agent_role`: Optional role/description
- `track_tokens`: Whether to track token usage (default: True)
- `track_costs`: Whether to estimate costs (default: True)

**Callback Methods:**
- `on_chain_start(serialized, inputs, **kwargs)`: Chain execution starts
- `on_chain_end(outputs, **kwargs)`: Chain completes successfully
- `on_chain_error(error, **kwargs)`: Chain fails with error
- `on_llm_start(serialized, prompts, **kwargs)`: LLM call starts
- `on_llm_end(response, **kwargs)`: LLM call completes
- `on_agent_action(action, **kwargs)`: Agent takes action (tool call)
- `on_agent_finish(finish, **kwargs)`: Agent completes reasoning
- `on_tool_start(serialized, input_str, **kwargs)`: Tool execution starts
- `on_tool_end(output, **kwargs)`: Tool execution completes
- `on_tool_error(error, **kwargs)`: Tool execution fails

### LangGraphMultiAgentMonitor

```python
class LangGraphMultiAgentMonitor:
    def __init__(
        self,
        client: WhiteBoxXAI,
        workflow_name: str,
        meta_data: Optional[Dict[str, Any]] = None
    )
```

**Methods:**

`start_monitoring(inputs: Optional[Dict[str, Any]] = None) -> str`
- Starts workflow monitoring
- Returns: workflow_id

`register_agent(agent_name: str, role: Optional[str], model_name: Optional[str], tools: Optional[List[str]], **kwargs) -> None`
- Registers an agent in the workflow

`get_callbacks(agent_name: str, agent_role: Optional[str] = None) -> List[BaseCallbackHandler]`
- Returns callbacks for a specific agent
- Reuses existing callback if called multiple times for same agent

`log_handoff(from_agent: str, to_agent: str, message: str, meta_data: Optional[Dict] = None) -> None`
- Logs an agent-to-agent handoff

`complete_monitoring(outputs: Optional[Dict[str, Any]] = None, status: str = "completed") -> Dict[str, Any]`
- Completes workflow and returns summary with analytics

### Helper Function

```python
def monitor_langchain_agent(
    client: WhiteBoxXAI,
    agent_executor: Any,
    workflow_name: str,
    agent_name: str = "main",
    inputs: Optional[Dict[str, Any]] = None,
    **run_kwargs
) -> Dict[str, Any]
```

**Returns:**
```python
{
    "result": Any,  # Agent execution result
    "workflow_id": str,  # Workflow ID for analytics
    "status": str,  # "completed" or "failed"
    "error": str  # Present only if status is "failed"
}
```

## Troubleshooting

### Issue: Callbacks not being called

**Solution:** Ensure you pass callbacks to the agent executor or graph invocation:

```python
# Correct
result = agent_executor.run(input_data, callbacks=[callback])

# Also correct
result = graph.invoke(inputs, config={"callbacks": [callback]})
```

### Issue: Missing token counts

**Solution:** Token tracking depends on LLM provider returning token usage. Some providers may not include this data. Check `llm_output` in LangChain's response:

```python
callback = MultiAgentCallbackHandler(
    client=client,
    workflow_id=workflow_id,
    agent_name="test",
    track_tokens=True  # May not work with all LLM providers
)
```

### Issue: Cost estimates seem incorrect

**Solution:** Cost estimation uses rough averages. For accurate costs, integrate directly with your LLM provider's billing:

```python
# Override cost calculation
actual_cost = get_cost_from_provider()
client.agent_workflows.create_execution(
    workflow_id=workflow_id,
    agent_name="my_agent",
    cost=actual_cost  # Use actual cost from provider
)
```

## Additional Resources

- [Multi-Agent Monitoring Overview](../sdk/multi-agent-monitoring.md)
- [Multi-Agent Quick Reference](../sdk/multi-agent-quick-reference.md)
- [CrewAI Integration Guide](../sdk/multi-agent-monitoring.md#crewai-integration)
- [REST API Documentation](../sdk/multi-agent-quick-reference.md#api-endpoints)
- [LangChain Documentation](https://python.langchain.com/docs/modules/callbacks/)
- [LangGraph Documentation](https://langchain-ai.github.io/langgraph/)

---

**Need Help?**

- GitHub Issues: https://github.com/whiteboxxai/whiteboxxai/issues
- Documentation: https://docs.whiteboxxai.com
- Email: support@whiteboxxai.com
