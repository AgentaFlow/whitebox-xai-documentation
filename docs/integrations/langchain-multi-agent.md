---
description: Record LangChain and LangGraph multi-agent systems in WhiteBoxXAI as workflows — agents, tasks, executions and agent-to-agent handoffs — alongside every LLM call, with per-agent cost, bottlenecks and a timeline.
---

# LangChain Multi-Agent

When several agents collaborate (a supervisor routing to specialists, a researcher handing off to a
writer, a LangGraph graph of agent nodes), a reviewer needs to see who did what, in what order, and at
what cost. WhiteBoxXAI records the run as a **multi-agent workflow**:

- the **agents** taking part, with their roles and models
- the **tasks** they worked on and how each ended
- each agent's **executions**, and the **handoffs** between agents
- every **LLM call** each agent made, through the [LangChain callback handler](langchain.md)

After the run, WhiteBoxXAI computes per-agent cost, bottlenecks and a timeline. Runs appear under
**Observability → Agents** in the dashboard. Each workflow is one
[Governance Event](../get-started/governance-events.md#agent-workflows-one-event-per-workflow),
however many agents and steps it has.

!!! tip "A shorter route: the helpers"
    This page uses `client.agent_workflows` so every call is visible. The helpers in
    `whiteboxxai.integrations.langchain_agents` wrap the same calls:

    ```python
    from whiteboxxai.integrations.langchain_agents import LangGraphMultiAgentMonitor

    monitor = LangGraphMultiAgentMonitor(client, workflow_name="claims-triage")
    workflow_id = monitor.start_monitoring(inputs={"claim_id": "CLM-2291"})
    monitor.register_agent("triage", role="Classifies the claim", model_name="gpt-4o")
    callbacks = monitor.get_callbacks("triage")       # pass as config={"callbacks": callbacks}
    monitor.log_handoff("triage", "fraud-check", message="Claim classified")
    monitor.complete_monitoring(outputs={"decision": "approve"})
    ```

    `MultiAgentCallbackHandler(client, workflow_id, agent_name="triage")` is the per-agent callback
    handler behind `get_callbacks`, and `monitor_langchain_agent(client, agent_executor, workflow_name)`
    runs a single agent as a one-agent workflow and returns its result and `workflow_id`.

## Install

```bash
pip install "whitebox-xai-sdk[langchain]" langgraph langchain-openai
```

## Record a run

```python
import os

from whiteboxxai import WhiteBoxXAI
from whiteboxxai.integrations.langchain import LangChainMonitor, WhiteBoxXAICallbackHandler

client = WhiteBoxXAI(api_key=os.environ["WHITEBOX_API_KEY"])
wf = client.agent_workflows

# 1. Create the workflow (this is the one Governance Event)
workflow = wf.create_and_start(name="claims-triage", framework="langchain",
                               meta_data={"ticket": "CLM-2291"})
workflow_id = workflow["id"]

# 2. Register the agents
triage = wf.register_agent(workflow_id, name="triage", role="Classifies the claim",
                           model_name="gpt-4o", llm_provider="openai")
fraud = wf.register_agent(workflow_id, name="fraud-check", role="Checks for fraud signals",
                          model_name="claude-sonnet-4-6", llm_provider="anthropic")

# 3. Start the run
wf.start(workflow_id, inputs={"claim_id": "CLM-2291"})

# 4. LLM calls from every agent are logged by the LangChain callback handler
handler = WhiteBoxXAICallbackHandler(LangChainMonitor(client, application_name="claims-triage"))
```

Then, while the graph or agents run, record what happens:

```python
# A task assigned to an agent
task = wf.create_task(workflow_id, task_name="Classify claim", agent_id=triage["id"],
                      input_data={"claim_id": "CLM-2291"})

# The agent starts work
execution = wf.create_execution(workflow_id, agent_id=triage["id"],
                                inputs={"claim_id": "CLM-2291"}, execution_order=1)

result = triage_agent.invoke({"messages": [("user", claim_text)]},
                             config={"callbacks": [handler]})

# The task's outcome
wf.update_task_status(task["id"], status="completed", output_data={"category": "water damage"})

# A handoff from one agent to another
wf.log_interaction(workflow_id, interaction_type="delegation",
                   from_agent_id=triage["id"], to_agent_id=fraud["id"],
                   from_execution_id=execution["id"],
                   message="Claim classified; check for duplicate submissions")
```

Finally, complete the workflow, including when it fails, so it isn't left running:

```python
wf.complete(workflow_id, status="completed", outputs={"decision": "approve"})
# or: wf.complete(workflow_id, status="failed", error_message=str(err))
```

Steps after `create_and_start` (agents, tasks, executions, interactions and completion) are free.

## With LangGraph

Record agents once, when you build the graph, and add the handoff in the node that routes between
agents:

```python
from langgraph.graph import StateGraph, END

def route(state):
    nxt = "fraud" if state["needs_fraud_check"] else END
    if nxt != END:
        wf.log_interaction(workflow_id, interaction_type="handoff",
                           from_agent_id=triage["id"], to_agent_id=fraud["id"],
                           message="Routing to fraud check")
    return nxt

graph = StateGraph(ClaimState)
graph.add_node("triage", triage_node)
graph.add_node("fraud", fraud_node)
graph.set_entry_point("triage")
graph.add_conditional_edges("triage", route)
app = graph.compile()

app.invoke({"claim": claim_text}, config={"callbacks": [handler]})
```

Interaction types used in practice include `delegation`, `handoff`, `message` and `tool_call`.

## Read the analytics

Analytics are computed after `complete` and may take a short while to be ready:

```python
wf.analytics(workflow_id)          # tokens, costs, execution counts, durations
wf.cost_breakdown(workflow_id)     # per agent
wf.bottlenecks(workflow_id)        # slowest agents and tasks, failing agents
wf.timeline(workflow_id)           # every event in order
```

`cost_breakdown`, `bottlenecks` and `timeline` can take up to about 30 seconds.

## See also

- [Multi-Agent Monitoring](../sdk/multi-agent-monitoring.md): concepts and the REST API
- [CrewAI](crewai.md)
- [LangChain](langchain.md): single-agent apps and the callback handler
