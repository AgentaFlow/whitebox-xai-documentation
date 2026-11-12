---
description: WhiteBoxXAI Observability — runtime performance for ML models, LLM & RAG audit trails (usage, cost, safety, retrieval quality, conversation log) and multi-agent workflow runs.
---

# Observability

**Observability** shows how the systems in your register behave at runtime. Its tabs are
**Performance**, **LLM & RAG**, **Agents** and [Evaluations](evaluations.md).

## Performance

For classic ML systems:

- **Platform health**: **Healthy**, **Degraded** or **Unhealthy**
- **System performance** over the selected range (**7 days**, **30 days**, **90 days** or **1 year**)
- **Baseline Performance Metrics**: accuracy, precision, recall, F1 score and AUC-ROC, as
  registered with the model, next to what has been measured since

Performance metrics you report from your own evaluation pipeline (`client.metrics.create`) appear here
too. They don't count as [Governance Events](../get-started/governance-events.md).

## LLM & RAG audit trails

For systems of type **LLM**, this tab is the audit trail of what your LLM application actually did:
every logged call, retrieval and safety scan, searchable after the fact. It's the record you need
when someone asks *what did the assistant tell that customer, using which prompt version, and was
the answer checked?*

| Panel | Shows |
| --- | --- |
| **Usage** | Requests, tokens and spend for the period |
| **Safety** | Toxicity and PII findings, and how many responses were blocked |
| **Retrieval quality** | Precision and relevance of RAG retrievals, where ground truth was supplied |
| **Conversation log** | Every logged prompt and completion, searchable (**Search conversations...**) |
| **RAG quality trend** | Retrieval quality over time |

Pick a range from 24 hours to 1 year. **Export Report** downloads the panel data.

### What the trail contains

Each logged LLM call records the provider, model, prompt, completion, token counts, latency and
status as your application reported them, plus the [prompt version](evaluations.md#prompt-versions)
it used if you pass one. WhiteBoxXAI doesn't estimate token counts. Spend is calculated from the
reported tokens for model names it has prices for, and is left blank for the rest. See
[LLM monitoring](../sdk/llm-monitoring.md) for which providers you can log from (any of them).

Safety scans check text for:

- **Toxicity**: toxicity, severe toxicity, identity attack, insult, profanity, threat, sexually
  explicit content, hate speech, violence, self-harm and misinformation
- **Personal data (PII)**: 12 types, including email addresses and phone numbers
- **Harmful content**

Flagged categories are recorded as they were found and are never downgraded after the fact.

**Retrieval quality** metrics are only computed when you supply ground-truth document ids with a
retrieval. Without them, the panel says the metrics haven't been computed.

### Attribution & explainability

When someone asks *why did the assistant say that, and was it grounded in the documents it
retrieved?*, the **LLM & RAG Attribution & Explainability** panel answers with a computation you
can re-run, not a summary. If you have more than one LLM system, choose a **Target System** first.
The fields start with a sample scenario; replace them with your own text.

| Tab | What you enter | What you get |
| --- | --- | --- |
| **RAG Grounding & Attribution** | A **User Query**, the **Generated Answer** and the retrieved context chunks. **Analyze RAG Grounding** runs it. | An **Auditor Verdict** in plain language, with the analysis method shown. **Sentence-by-Sentence Claim Grounding** marks each claim **Grounded** or **Ungrounded Claim**, with its supporting chunk and similarity. **Context Chunk Attribution Weights** show how much each chunk contributed. |
| **Token Attribution** | An **Input Prompt** and the **Model Completion**, and a **Method**: **Integrated Gradients (Path Integral)**, **Input × Gradient** or **Leave-One-Out Perturbation**. **Compute Token Importance** runs it. | A verdict on the method used, a heat strip over the tokens, and a table of each token's importance score, labelled **Salient Driver** or **Contextual**. |
| **Prompt Robustness & Ablation** | The **Prompt for Ablation Testing**, and an **Ablation Mode**: **Sentence Ablation (Deterministic)**, **Clause Removal** or **Character Perturbation**. **Analyze Sensitivity** runs it. | A **Robustness & Integrity Status**: **Measured Divergence**, or an explicit **Unmeasured** state. |

The panel never fills a gap with an invented number. When a result can't be measured, it says
**Unmeasured** and why. An ungrounded claim is a claim no retrieved chunk supports. It is not proof
the claim is false, so treat it as a lead for review.

The same analyses are available from code and the API. See [LLM monitoring](../sdk/llm-monitoring.md#llm-explainability)
and the [REST API Reference](../sdk/api-reference.md#llm-explainability).

## Agents

Multi-agent workflow runs from CrewAI, LangChain/LangGraph, AutoGen, n8n or your own framework.
Filter by **status** (Running, Completed, Failed, Pending, Cancelled) and **framework** (CrewAI,
LangChain, AutoGen, n8n, Custom). Open a run to see its agents, tasks, executions, agent-to-agent
interactions, cost breakdown, bottlenecks and timeline.

Cost, bottleneck and timeline analytics are computed after a workflow completes, and can take a
little while to appear.

Each workflow counts as one [Governance Event](../get-started/governance-events.md), however many
agents and steps it has. See [Multi-Agent Monitoring](../sdk/multi-agent-monitoring.md).

## Evaluations

Prompt evaluations, golden datasets, regression checks and prompt versions have their own page:
[Evaluations & Prompt Versions](evaluations.md).
