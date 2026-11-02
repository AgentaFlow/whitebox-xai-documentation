---
description: Monitor CrewAI crews with WhiteBoxXAI — register agents and tasks automatically, log executions, delegations and task results, and get cost, bottleneck and timeline analytics, as one Governance Event per run.
---

# CrewAI

`CrewAIMonitor` records a CrewAI crew run as a WhiteBoxXAI multi-agent workflow: the crew's agents and
tasks, each agent's executions, delegations between agents, task results, and the final outcome. After
the run you get per-agent cost, bottlenecks and a timeline, under **Observability → Agents** in the
dashboard.

Each crew run is one [Governance Event](../get-started/governance-events.md#agent-workflows-one-event-per-workflow),
however many agents, tasks and steps it has.

## Install

```bash
pip install whitebox-xai-sdk crewai
```

## Monitor a crew

```python
import os

from crewai import Agent, Crew, Process, Task
from whiteboxxai.integrations.crewai_monitor import monitor_crew

researcher = Agent(role="Research Analyst", goal="Find accurate information",
                   backstory="Careful researcher who cites sources")
writer = Agent(role="Content Writer", goal="Write a clear summary",
               backstory="Writes for a non-technical audience")

research = Task(description="Research the EU AI Act's obligations for deployers",
                expected_output="A list of obligations with article references", agent=researcher)
summary = Task(description="Summarise the research for a risk committee",
               expected_output="A one-page summary", agent=writer)

crew = Crew(agents=[researcher, writer], tasks=[research, summary], process=Process.sequential)

monitor = monitor_crew(
    crew=crew,
    workflow_name="EU AI Act deployer briefing",
    api_key=os.environ["WHITEBOX_API_KEY"],
    metadata={"requested_by": "risk-committee"},
)

try:
    result = crew.kickoff()
    monitor.complete_monitoring(outputs={"summary": str(result)})
except Exception as err:
    monitor.complete_monitoring(status="failed", error_message=str(err))
    raise
```

`monitor_crew` creates the workflow and registers every agent and task in the crew. Always call
`complete_monitoring`, including when the run fails, so the workflow isn't left running.

On Business Cloud, pass your tenant's API address as `api_url=`.

## Add detail during the run

Call these from CrewAI callbacks or your own code to record what happened inside the run:

```python
monitor.log_agent_execution(researcher, inputs={"query": "..."}, outputs={"findings": "..."},
                            tokens_used=1840, cost=0.012)
monitor.log_interaction(researcher, writer, interaction_type="delegation",
                        message="Handing over research notes")
monitor.log_task_completion(research, status="completed", output_data={"obligations": 14})
```

These are free; only creating the workflow counts as a Governance Event. Report token counts and
costs your LLM provider actually returned, or leave them out.

## Read the analytics

Analytics are computed after `complete_monitoring` and can take a short while to be ready:

```python
analytics = monitor.get_analytics()
```

Or from the SDK client, with the workflow id `monitor.workflow_id`:

```python
client.agent_workflows.cost_breakdown(workflow_id)
client.agent_workflows.bottlenecks(workflow_id)
client.agent_workflows.timeline(workflow_id)
```

## See also

- [Multi-Agent Monitoring](../sdk/multi-agent-monitoring.md): concepts, the REST API and other frameworks
- [LangChain Multi-Agent](langchain-multi-agent.md)
- [LLM monitoring](../sdk/llm-monitoring.md): log the individual LLM calls your agents make
