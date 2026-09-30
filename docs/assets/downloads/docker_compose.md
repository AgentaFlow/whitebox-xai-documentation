# Skill: Docker & Local Environment Management

## Overview

How the local development stack actually starts, and how to fix the container issues that come up.

## 1. What `./start-dev.sh` really does

**It is not a full container stack.** `start-dev.sh` runs the backend (uvicorn) and the frontend
(npm) **natively on your host**, and brings up **only PostgreSQL** through docker-compose. Use it
for day-to-day work:

```bash
./start-dev.sh                  # backend + frontend natively, Postgres in Docker
./start-dev.sh --backend-only
./start-dev.sh --frontend-only
./start-dev.sh --skip-install   # skip dependency install
./start-dev.sh --stop-db        # stop the Postgres container on exit
./start-dev.sh --help
```

It lives at the repo root (its own `--help` text says `./scripts/start-dev.sh`; that path is wrong).

## 2. The full container stack

`docker-compose.yml` defines **six** services — backend, frontend, postgres, redis, celery-worker,
mcp-server:

```bash
docker compose up -d                    # everything
docker compose up -d postgres redis     # just the backing services
docker compose up --build -d            # rebuild after dependency changes
docker compose logs -f backend
docker compose down                     # add -v to drop the postgres_data/redis_data volumes
```

`docker compose` (v2, space) is current; older `docker-compose` still works and is what
`start-dev.sh` shells out to.

## 3. Permission errors (Linux / WSL only)

On Windows with Docker Desktop these do not apply — restart Docker Desktop instead. On Linux or
inside WSL, `start-dev.sh` already calls `scripts/ensure-docker-permissions.sh` on startup. If a
socket or permission error still appears:

```bash
./scripts/fix-docker-permissions.sh
# or, manually:
sudo chgrp docker /var/run/docker.sock && sudo chmod g+rw /var/run/docker.sock
newgrp docker        # or open a new shell -- group changes need a fresh login
```

## 4. Notes

- Redis is optional for the unit test suite: tests log connection errors and pass, and rate limiting
  fails open without it.
- The integration suite needs a real Postgres — it is the only thing in CI that runs the Alembic
  migration chain. Bring one up with `docker compose up -d postgres` before
  `pytest tests/ -m integration`.
- Disk usage grows fast on Windows (the Docker Desktop VHDX does not shrink on its own). Prune
  periodically: `docker system prune -a --volumes` — this deletes unused images and volumes, so
  stop the stack first and be sure you do not need the local database.
