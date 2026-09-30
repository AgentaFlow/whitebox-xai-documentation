# Skill: Python & Backend Development Strategy

## Overview

Standard operating procedures for the FastAPI backend, Python dependencies, database migrations,
and virtual environments in WhiteBoxXAI. Repo-wide rules live in [`/AGENTS.md`](../../AGENTS.md);
this file is the how-to.

## Prerequisites

- **Python 3.11+** — the system interpreter on the dev machine is 3.9 and **cannot import the
  backend at all** (`datetime | None` syntax fails). This is the single most common false "code bug".
- **FastAPI** 0.141.1, **SQLAlchemy** + **Alembic**, **Celery** + **Redis**.

## 1. Environment setup

`.venv-test/` is this repo's conventional dev virtualenv. **It is gitignored (`.gitignore:284`), so
a fresh clone does not have one.** Create it first:

```bash
# Windows (PowerShell or Git Bash)
py -3.11 -m venv .venv-test
.venv-test/Scripts/python.exe -m pip install --upgrade pip
.venv-test/Scripts/python.exe -m pip install -r requirements-dev.txt
.venv-test/Scripts/python.exe -m pre_commit install

# Linux / macOS
python3.11 -m venv .venv-test
.venv-test/bin/python -m pip install --upgrade pip
.venv-test/bin/python -m pip install -r requirements-dev.txt
.venv-test/bin/python -m pre_commit install
```

Then confirm it works — this is the check that catches a 3.9 interpreter:

```bash
V=.venv-test/Scripts/python.exe          # Windows
# V=.venv-test/bin/python                # Linux/macOS
$V --version                             # expect 3.11.x
$V -c "import backend.main; print('backend imports OK')"
```

The name is a convention, not a requirement — every `$V` in this repo's docs means "a Python 3.11
interpreter with `requirements-dev.txt` installed". Only `.vscode/settings.json`
(`python.defaultInterpreterPath`) hard-codes the path; point it at yours if you use a different one.

**`pre-commit install` is not optional.** The four gates in section 5 run as pre-commit hooks *and*
CI jobs. Without the hook installed you only discover a violation after opening the PR — that is how
formatting errors have reached `main` before.

The three requirements files nest, so `requirements-dev.txt` gets you everything:

| File | Contents |
|---|---|
| `requirements-prod.txt` | Runtime only — what ships in the production container |
| `requirements.txt` | `-r requirements-prod.txt` + heavy ML frameworks (torch, tensorflow, …) |
| `requirements-dev.txt` | `-r requirements.txt` + **pytest, coverage, lint and docs tooling** |

**`requirements.txt` does not contain pytest.** Installing it alone and then running `pytest` fails
with "command not found", which looks like a broken checkout. Note also that several transitive pins
in these files are deliberate (`tokenizers==0.15.2` against `transformers==4.35.2`, a `numba`
constraint for coverage) — read the comments before loosening one.

## 2. Running the test suite

```bash
$V -m pytest
```

**Pass no coverage flags.** `pytest.ini` already sets `--cov=backend`,
`--cov-report=html:coverage_html`, `-m "not smoke"`, and — the one that fails CI —
**`--cov-fail-under=79`**. Adding `--cov-report=html` on the command line silently redirects the
report away from `coverage_html/`.

```bash
$V -m pytest tests/unit                  # fast loop
$V -m pytest tests/ -m integration       # Postgres; the ONLY job that runs migrations
$V -m pytest -m smoke                    # opt-in, hits a real deployed environment
```

## 3. Local server

```bash
$V -m uvicorn backend.main:app --reload --port 8000
```

Redis is optional locally — tests log connection errors and pass, and rate limiting fails open.
See [`docker_compose.md`](docker_compose.md) for the database container and the full local stack.

## 4. Database migrations

Single head. **A green pytest run says nothing about schema correctness** — the unit suite builds
its schema from `Base.metadata.create_all` on SQLite and never executes the migration chain. Always
round-trip against a real Postgres:

```bash
alembic revision --autogenerate -m "description_of_changes"
alembic upgrade head && alembic downgrade -1 && alembic upgrade head
```

Use `backend.models.database.UUID(length=36)` for UUID columns, add
`values_callable=lambda e: [m.value for m in e]` to every `SQLEnum`, and register new models in
`backend/models/__init__.py` — see the schema section of [`/AGENTS.md`](../../AGENTS.md) for why
each of those has already caused a production defect.

## 5. What CI actually enforces

Run this before every PR — it is the same set of hooks CI runs as blocking jobs:

```bash
pre-commit run --all-files
```

That covers black (88 cols), isort, flake8, bandit, and the four repo gates:

| Gate | Script | Trips when |
|---|---|---|
| New modules need tests | `scripts/check_new_modules_have_tests.py` | A new file under `backend/services\|repositories\|api/v1`, `sdk/whiteboxxai/`, or `mcp_server/whiteboxxai_mcp/` has no matching test in the same commit |
| Routers registered | `scripts/check_routers_registered.py` | A module under `backend/api/` defines `router` but is not in `backend/api/v1/__init__.py` |
| Schema lock | `scripts/check_schema_lock.py` | `.github/schema-lock.json` is locked and the PR touches `alembic/versions/`, `backend/models/`, or `database/` without the override label |
| Marketing features shipped | `scripts/check_marketing_features_shipped.py` | A pillar in the marketing feature grid has no live backend router |

Plus the 79% coverage floor and a Conventional Commits PR title.

## 6. Verification one-liners

```bash
# Is the API surface reachable? (app.routes holds lazy wrappers as of FastAPI 0.141.1)
$V -c "from backend.main import app; from fastapi.routing import iter_route_contexts; \
       print(len({c.path for c in iter_route_contexts(app.routes)}), 'paths')"

# Does every module import? (catches dead code that name-grepping misses)
$V -c "
import importlib, pkgutil
for m in ('backend.services','backend.repositories','backend.api.v1','backend.schemas','backend.models'):
    pkg = importlib.import_module(m)
    for _, n, _ in pkgutil.iter_modules(pkg.__path__):
        try: importlib.import_module(f'{m}.{n}')
        except Exception as e: print('BROKEN', f'{m}.{n}', e)
"

# Do all mappers configure? (catches relationship/column mismatches)
$V -c "from sqlalchemy.orm import configure_mappers; import backend.models; configure_mappers(); print('ok')"
```
