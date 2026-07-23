# WhiteBoxXAI Documentation

This repository contains the source for the **WhiteBoxXAI user documentation site**,
published with [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/) at:

**➡️ [docs.whiteboxxai.com](https://docs.whiteboxxai.com)**

WhiteBoxXAI is the AI observability, explainability, and governance platform by
[AgentaFlow](https://whiteboxxai.com). The documentation here helps users monitor their
models, explain predictions, detect drift, audit for bias, and stay compliant.

## What's here

```
docs/            # Documentation content (Markdown)
  index.md       # Home page
  get-started/   # Getting Started, Plans & Limits
  user-guide/    # Using the platform, features, governance
  sdk/           # Python SDK and REST API reference
  integrations/  # Framework and tooling integrations
  account/       # Account security & compliance
  learn/         # Case studies, workshops, demos, certification
  help/          # FAQ and troubleshooting
mkdocs.yml       # Site configuration and navigation
```

## Preview locally

You'll need Python 3.9+.

```bash
# 1. (optional) create a virtual environment
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate

# 2. install the docs toolchain
pip install -r requirements-docs.txt

# 3. serve with live reload at http://127.0.0.1:8000
mkdocs serve
```

To produce the static site under `site/`:

```bash
mkdocs build --strict
```

The `--strict` flag fails the build on broken internal links, which keeps the published
site clean.

## How it's published

Every push to `main` triggers the [Deploy documentation](.github/workflows/deploy-docs.yml)
GitHub Actions workflow, which builds the site (strict) and publishes it to the `gh-pages`
branch. GitHub Pages serves that branch at the custom domain `docs.whiteboxxai.com`
(configured via `docs/CNAME`).

## Contributing

- Edit or add Markdown files under `docs/`.
- Add new pages to the `nav:` tree in `mkdocs.yml`.
- Run `mkdocs serve` to preview, and `mkdocs build --strict` before opening a pull request.

This is **public, end-user documentation**. Please do not add internal material —
architecture and infrastructure details, deployment or operations runbooks, environment
variables, secrets, or anything else that shouldn't be shared publicly.

## Support

- **Documentation:** [docs.whiteboxxai.com](https://docs.whiteboxxai.com)
- **Python SDK:** [pypi.org/project/whitebox-xai-sdk](https://pypi.org/project/whitebox-xai-sdk/)
- **Email:** [support@whiteboxxai.com](mailto:support@whiteboxxai.com)
