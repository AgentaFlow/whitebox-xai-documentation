---
description: The seven public WhiteBoxXAI agent skills — what each covers, when to load it, and how to install them into Claude Code, Copilot, Cursor or any harness that reads SKILL.md files.
---

# Agent Skills

Agent skills are `SKILL.md` files that teach a coding agent how to use WhiteBoxXAI correctly: which
SDK calls and MCP tools exist, what their arguments mean, and what the agent must never do (invent
a score, commit a key, cast a governance vote). Each skill has a `TRIGGER` / `SKIP` description so
the agent loads it only when it applies.

All skills are MIT-licensed, published by AgentaFlow, and checked against the current SDK and MCP
tool list before release.

## Install

Pick one:

=== "Paste into your agent"

    ```text
    Read https://whiteboxxai.com/.well-known/skills/whitebox-getting-started/SKILL.md and set up WhiteBoxXAI for this repository.
    ```

=== "Skills CLI"

    ```bash
    npx skills add https://whiteboxxai.com
    ```

    Lists every skill and installs the ones you pick.

=== "Zip download"

    Download [`whitebox-skills.zip`](https://whiteboxxai.com/skills/whitebox-skills.zip) and unzip it
    into your agent's skills directory, for example `.claude/skills/` or `.agents/skills/`.

Harnesses that support skill discovery can read the index at
`https://whiteboxxai.com/.well-known/skills/index.json`. Each entry carries a SHA-256 digest so the
harness can verify the file it downloads.

## The skills

**Always load `whitebox-governance-rules` alongside any other WhiteBoxXAI skill.**

| Skill | Load it when the user wants to... | Skip it when... |
|---|---|---|
| [`whitebox-getting-started`](https://whiteboxxai.com/.well-known/skills/whitebox-getting-started/SKILL.md) | Add or connect WhiteBoxXAI: install the SDK or MCP server, set an API key, choose an integration path | WhiteBoxXAI is already configured |
| [`whitebox-governance-rules`](https://whiteboxxai.com/.well-known/skills/whitebox-governance-rules/SKILL.md) | Do anything that reads from or writes to WhiteBoxXAI | The work never touches WhiteBoxXAI |
| [`whitebox-register-and-monitor-models`](https://whiteboxxai.com/.well-known/skills/whitebox-register-and-monitor-models/SKILL.md) | Register and log predictions for scikit-learn, XGBoost, LightGBM, PyTorch, TensorFlow/Keras or Hugging Face models | The system is an LLM, RAG or agent app |
| [`whitebox-drift-alerts-explainability`](https://whiteboxxai.com/.well-known/skills/whitebox-drift-alerts-explainability/SKILL.md) | Check drift, set alert rules, explain predictions with SHAP/LIME, run a fairness audit | The model isn't registered yet |
| [`whitebox-llm-and-agent-observability`](https://whiteboxxai.com/.well-known/skills/whitebox-llm-and-agent-observability/SKILL.md) | Log LLM calls, RAG retrievals, safety checks and multi-agent workflows; version, deploy and roll back prompts; group a run into one Governance Event | The system is a tabular ML model |
| [`whitebox-mcp-server`](https://whiteboxxai.com/.well-known/skills/whitebox-mcp-server/SKILL.md) | Connect an assistant, IDE or agent harness over MCP, or pick the right MCP tool | Instrumenting application code (use the SDK skills) |
| [`whitebox-compliance-evidence`](https://whiteboxxai.com/.well-known/skills/whitebox-compliance-evidence/SKILL.md) | Summarise ISO/IEC 42001 readiness, the risk register, review dossiers, or reports for a risk officer or auditor | The request needs a legal conclusion |

The full index with descriptions is at
[`whiteboxxai.com/skills/index.md`](https://whiteboxxai.com/skills/index.md).

## What the skills enforce

The skills say more than "how to call the API". They also carry the rules a compliance team
depends on:

- Report empty, errored or unassessed results as they are, never as a default value.
- Put a plain-language verdict beside every drift statistic, fairness metric or explanation weight.
- Read keys from the environment; never write one into source, notebooks or logs.
- Treat a `404` on an id as "not in this organization", not as an outage.
- Never cast a governance vote on a person's behalf.
- Stay within ISO/IEC 42001, GDPR, CCPA, the EU AI Act and NIST AI RMF.

See [Agent & Developer Resources](index.md) for the other agent entry points.
