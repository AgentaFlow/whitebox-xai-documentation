---
description: Prompt evaluations in WhiteBoxXAI — golden datasets, scoring methods, evaluation runs, regression checks between prompt versions, and governed prompt versions with a design → verify → deploy → retired lifecycle.
---

# Evaluations & Prompt Versions

A prompt change can quietly make an LLM application worse, and a regulator or customer can ask you
to show it didn't. **Observability → Evaluations** gives you the evidence: golden datasets of
expected behaviour, scored runs against a specific prompt version, and a regression check between
two runs. Prompts are versioned and governed like models.

!!! note "WhiteBoxXAI does not call your model"
    Evaluations score outputs your model *already produced*, stored with each dataset row or
    reported by your own test pipeline. WhiteBoxXAI never sends prompts to your model or to any LLM
    provider on your behalf, and it doesn't use another model to judge results.

The tab has three parts: **Runs**, **Golden datasets** and **Prompts**.

## Golden datasets

A golden dataset is *"a set of inputs with the output you expect for each."*

1. **New dataset**: give it a name and, optionally, what it's for.
2. **Add a row** for each case:
    - **Input** (JSON object): what was sent to the model
    - **Expected output** (JSON object, optional): what a correct answer looks like
    - **Stored model output** (JSON object, optional): *"The output your model gave for this
      input. A run scores this; without it the row is not evaluated."*

Rows without a stored output are shown as **Not evaluated** in every run. They're never counted as
passes.

## Score a prompt version

**Score a prompt version** creates an evaluation run for one prompt version against one golden
dataset. Choose **How to score**:

| Method | What it checks |
| --- | --- |
| **Exact match** | *"Compare each stored output with the row's expected output."* |
| **Required phrases** | *"Check each stored output for the phrases in the row's must_contain list."* |
| **Rule-based checks** | *"Compare expected values, and apply the no-PII rule if the rubric names it."* You can add a rubric and the score a row must reach to pass (0 to 1). |
| **Scored by your pipeline** | Results your own test pipeline or the SDK reported. *"WhiteBox recorded them as sent."* |

Results read as **N of M evaluated rows passed**, with **Not evaluated** rows listed separately.
Every run records **How it was scored**. A run that didn't record its method says so rather than
guessing.

Each evaluation run started through the API counts as one
[Governance Event](../get-started/governance-events.md). Authoring datasets and prompts is free.

## Check a prompt change for regressions

**Check a prompt change for regressions** compares two runs on the same dataset:

- **Baseline run**: the version in use
- **Candidate run**: the change

The result is one of three verdicts:

- **No regression found**
- **Regression found**, with the **Cases that got worse**
- **Cannot tell**, for example when too few rows were judged in both runs

The comparison lists rows that **Got better**, **Got worse** or stayed **Unchanged**, and which were
**Left out** because they weren't judged in one or both runs. Both runs must use the same dataset.

## Prompt versions

Under **Prompts**, **New prompt** creates a prompt template. Open it to see its versions, newest
first, and **Save a new version**. A version records:

- **User message** and **System message**
- **Target model** and **Temperature setting**
- **Git commit**: the code it shipped with
- **Tool schemas**
- **Version tag**

Versions **can't be edited or deleted**. A change is always a new version, so the history is
complete.

### Lifecycle

Each version moves through a governed lifecycle:

**design → verify → deploy → retired**

The **Lifecycle Stage** badge shows where a version is. **Verified** (Benchmark Verification) shows
that it passed verification, and **Active** marks the version in use. A version can't be deployed
without a passed verification. If a deployed version misbehaves, roll back to an earlier one, with
a recorded reason.

Lifecycle changes, deployment and rollback are done from the SDK or MCP server:

```python
v = client.prompts.create_version(template_id, user_message_template="...", version_tag="v3")
client.prompts.advance_stage(template_id, v["id"], target_stage="verify")
client.prompts.deploy(template_id, v["id"])          # refused without a passed verification
client.prompts.rollback(template_id, target_version_id=previous_id, reason="tone regression")
```

### Link calls to a version

Pass `prompt_version_id` when you log LLM calls. Every call in the
[LLM & RAG audit trail](observability.md#llm-rag-audit-trails) is then attributable to the exact
prompt that produced it, and prompt drift reports and incidents can be traced to a version:

```python
client.llm.log_call(
    provider="openai",
    model_name="gpt-4o",
    prompt=rendered_prompt,
    completion=answer,
    latency_ms=640.0,
    prompt_version_id=active_version_id,
)
```

See [LLM monitoring](../sdk/llm-monitoring.md) and the `prompts` tools on the
[MCP Server](../integrations/mcp.md).
