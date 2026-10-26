---
description: Machine-readable entry points for AI agents and coding assistants working with WhiteBoxXAI — llms.txt, agent skills, the MCP server, prompts and client configs.
---

# Agent & Developer Resources

WhiteBoxXAI is used by people and by the AI agents working for them. This section collects the
entry points an agent needs to set up WhiteBoxXAI, call it correctly, and report what it returned
without inventing anything.

## Start here (for agents)

| Resource | URL | Use it to |
|---|---|---|
| Docs index for LLMs | [`https://docs.whiteboxxai.com/llms.txt`](https://docs.whiteboxxai.com/llms.txt) | Find the right docs page without crawling the whole site |
| Agent skills index | [`https://whiteboxxai.com/skills/index.md`](https://whiteboxxai.com/skills/index.md) | Pick which skill to load for a task |
| Skills discovery (JSON) | [`https://whiteboxxai.com/.well-known/skills/index.json`](https://whiteboxxai.com/.well-known/skills/index.json) | Auto-discover and verify skills in a harness |
| All skills (zip) | [`https://whiteboxxai.com/skills/whitebox-skills.zip`](https://whiteboxxai.com/skills/whitebox-skills.zip) | Install every skill at once |
| MCP server | [MCP Server](../integrations/mcp.md) | Give an assistant live tools over the user's WhiteBoxXAI data |
| Compliance-assistant prompt | [AI Prompts](prompts.md) | Bootstrap an assistant that summarises evidence for a risk officer |
| Client config files | [Agent Config Definitions](yaml-definitions.md) | Copy-paste MCP client configuration |

The fastest way to get a coding agent going is to paste this instruction into it:

```text
Read https://whiteboxxai.com/.well-known/skills/whitebox-getting-started/SKILL.md and set up WhiteBoxXAI for this repository.
```

## Which integration path?

| You are... | Use | Docs |
|---|---|---|
| Instrumenting Python code that trains or serves a model | Python SDK | [SDK guide](../sdk/index.md) |
| Instrumenting an LLM app, RAG pipeline or agent | Python SDK | [LLM monitoring](../sdk/llm-monitoring.md), [Multi-agent monitoring](../sdk/multi-agent-monitoring.md) |
| An assistant that should *query or act on* WhiteBoxXAI for a user | MCP server | [MCP Server](../integrations/mcp.md) |
| Working in a language other than Python | REST API under `/api/v1/` | [API Reference](../sdk/api-reference.md) |

## Rules for agents

These are the same rules the `whitebox-governance-rules` skill enforces. They exist because
WhiteBoxXAI output becomes compliance evidence that a person may hand to an auditor.

1. **Never invent a number.** If a call returns nothing, an error, or an unassessed state, say
   exactly that. A wrong score presented as a result is worse than no score.
2. **Put a plain-language verdict next to every ML number**, using the platform's own verdict when
   it returns one. Don't present your own pass/fail threshold as the platform's.
3. **Keep secrets out of code.** Read the key from the environment (`WHITEBOX_API_KEY` for the SDK,
   `WHITEBOXXAI_MCP_API_KEY` for the MCP server). Keys look like `wbx_live_...`.
4. **A 404 means "not in your organization".** Don't retry with other credentials or guess ids.
5. **Human decisions stay human.** Governance votes need a person's own session, two-factor
   authentication and an authenticator code. An organization API key is refused. Prepare and
   summarise the dossier; never vote on someone's behalf.
6. **Stay inside the five frameworks**: ISO/IEC 42001, GDPR, CCPA, the EU AI Act and NIST AI RMF.
   WhiteBoxXAI records evidence; it does not certify compliance.
7. **Meter responsibly.** Group one AI run's records with `governance_run()` and honour
   `Retry-After` on a `429`. See [Governance Events](../get-started/governance-events.md).
