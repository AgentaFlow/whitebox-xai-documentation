# Skill: Node.js & Frontend Development Strategy

## Overview

Standard operating procedures for the Next.js frontend. Repo-wide rules live in
[`/AGENTS.md`](https://github.com/AgentaFlow/whitebox-xai-azure/blob/main/AGENTS.md); this file is the how-to.

## Prerequisites

- **Node.js 22+**, **npm 10+** (`frontend/package.json` declares both in `engines`; CI pins Node 22).
- Next.js 15.5 (App Router), React 19, TypeScript **strict**, Tailwind, Shadcn/ui, TanStack Query.

## 1. Install and run

```bash
cd frontend
npm install
npm run dev
```

Note `npm ci` needs a lockfile in sync with `package.json`; if it fails on a fresh clone, use
`npm install` once and commit the resulting lockfile change separately.

## 2. Before every PR — run all four

CI runs each of these as a blocking step, and `npm run build` alone will not catch the first three:

```bash
cd frontend
npm run lint          # ESLint
npm run type-check    # tsc --noEmit
npm test              # vitest run, includes accessibility tests
npm run build         # production build
```

## 3. Clearing the Next.js cache

If the build fails in a way that looks stale:

```bash
rm -rf frontend/.next     # from the repo root
# or, if you are already inside frontend/:  rm -rf .next
```

## 4. Rules that will fail your PR

**All backend calls go through `frontend/lib/api-client.ts`.**

```ts
import { apiClient } from "@/lib/api-client";
```

It attaches the auth token, falls back to the httpOnly session cookie, adds the `X-Requested-With`
header `CSRFMiddleware` requires, and handles 401s. A hand-rolled `fetch` in a page component looks
like it works and then breaks — that is how seven governance pages shipped
`Authorization: Bearer null`. If `apiClient` has no method for your endpoint, add one there.

**Two tests read dashboard pages as text off disk**, so they fail on file *moves* rather than
behaviour. Update them in the same commit as any dashboard page move or rename:

- `frontend/lib/no-fabricated-metrics.test.ts` — `PAGES_TO_SCAN` throws a hard ENOENT on a moved
  path. It also enforces the no-invented-numbers rule below.
- `frontend/lib/alerts-query-keys.test.ts` — asserts on exact `queryKey` source strings; even a
  whitespace reformat breaks it.

**Never render a fabricated metric.** This is a compliance product: a number with no real backend
value behind it becomes a false audit record. Use an empty state or an explicit "not computed"
instead, and pair every ML artifact with a plain-language verdict — the buyer is a risk officer,
not an ML engineer.
