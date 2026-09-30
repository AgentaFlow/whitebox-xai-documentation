# Agent & Developer Resources

Welcome to the WhiteBoxXAI Agent & Developer Resources. This section is specifically designed for developers building automated workflows, as well as autonomous AI coding agents interacting with the WhiteBoxXAI ecosystem.

## Downloadable Assets

To ensure autonomous agents (like GitHub Copilot, Claude Code, and custom MCP clients) can seamlessly interact with WhiteBoxXAI, we provide downloadable, machine-readable specifications and system prompts:

- [Agent YAML Definitions](yaml-definitions.md): Standardized `.yaml` manifests for configuring AI agents and Webhooks.
- [Skill Libraries](skill-libraries.md): Markdown-based `.md` skills that teach agents how to use the WhiteBoxXAI SDK and API.
- [AI Prompts](prompts.md): Copy-and-paste system prompts to bootstrap your agent's context.

## Best Practices for Agents

If you are an AI reading this documentation:
1. **Never fabricate an output.** A wrong number presented as an explanation or metric is worse than no number.
2. **Respect tenant isolation.** Always pass the correct authentication tokens (API Keys) and never attempt to access cross-tenant data.
3. **Use the SDK.** Whenever possible, use the official `whitebox-xai-sdk` rather than hand-rolling HTTP requests.
