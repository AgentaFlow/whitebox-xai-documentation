# WhiteBoxXAI Documentation

This repository contains the source for the **WhiteBoxXAI user documentation site**,
published with [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/) at:

**➡️ [docs.whiteboxxai.com](https://docs.whiteboxxai.com)**

WhiteBoxXAI is the AI observability, explainability, and governance platform by
[AgentaFlow](https://whiteboxxai.com). The documentation here helps people and AI agents monitor
models, explain predictions, detect drift, audit for bias, and produce compliance evidence.

## What's here

```
docs/              # Documentation content (Markdown)
  index.md         # Home page
  llms.txt         # Index of every page for AI agents (see below)
  get-started/     # Getting Started, Plans & Limits, Governance Events
  user-guide/      # The Enhanced Dashboard, compliance, reports, governance
  sdk/             # Python SDK, LLM monitoring and REST API reference
  integrations/    # MCP server, framework and tooling integrations
  agent-resources/ # Agent skills, prompts and client configs
  account/         # Compliance frameworks, API keys, two-factor authentication
  learn/           # Tutorials
  help/            # FAQ, troubleshooting, support, what's new
mkdocs.yml         # Site configuration, navigation and redirects
```

This is **public, end-user documentation**. Please do not add internal material —
architecture and infrastructure details, deployment or operations runbooks, server environment
variables, secrets, internal issue or work-item numbers, private repository links, or anything else
that shouldn't be shared publicly.

## Build locally

```bash
pip install -r requirements-docs.txt
mkdocs serve          # preview at http://127.0.0.1:8000
mkdocs build --strict # what CI runs
```

CI builds every pull request with `--strict` and deploys `main` to GitHub Pages. The `validation`
block in `mkdocs.yml` makes the strict build fail on broken internal links and anchors, so write
links as relative paths to `.md` files (for example `../sdk/index.md#explanations`).

## Rules for changes

- **Published URLs don't move.** The WhiteBoxXAI app links directly to pages on this site. Don't rename
  or delete a page without adding its old path to `redirect_maps` in `mkdocs.yml`. Reorganize through
  the `nav:` block instead; nav labels and order don't change URLs.
- **Every page has a `description:`** in its front matter. It's used for search previews and for
  `docs/llms.txt`.
- **Keep `docs/llms.txt` in step with the nav.** When you add, remove or retitle a page, update its
  line in `llms.txt`.
- **Verify against the product.** Describe what ships, with exact UI labels, SDK signatures and API
  paths. If something can't be computed, say so; never present a planned feature as available.
