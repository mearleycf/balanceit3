# BalanceIt3 Agent Guide

## How To Start Work

To begin building this project, invoke the orchestrator agent:

```
Use the ai-team-orchestrator agent to begin execution of docs/plans/2026-04-08-balanceit3-full-development.md
```

The orchestrator reads this file and the implementation plan, then dispatches the right specialist agents per task.

## Agent Roles

| Agent | Responsibility |
|-------|---------------|
| `ai-team-orchestrator` | **Start here.** Reads plan, dispatches specialists, runs Council of 5 at phase boundaries, manages approval gates |
| `ai-team-backend-developer` | FastAPI routes, SQLAlchemy models, Alembic migrations, Python domain logic |
| `ai-team-frontend-developer` | Astro pages, React islands, Tailwind/shadcn/ui components |
| `ai-team-devops` | Vercel + Railway/Render deployment, CI/CD, env vars |
| `ai-team-dev-deploy-specialist` | Deploy artifact prep, rollout guidance |
| `ai-team-qa` | Test strategy, pytest + Vitest + Playwright suites |
| `ai-team-ux-designer` | UX flows, component design specs |
| `ai-team-api-implementation-specialist` | FastAPI contract design, Pydantic schemas |
| `ai-team-product-acceptance-reviewer` | FR coverage validation (Council of 5) |
| `ai-team-architecture-reviewer` | Layer boundary enforcement (Council of 5) |
| `ai-team-code-quality-reviewer` | Code clarity and hygiene (Council of 5) |
| `ai-team-test-reliability-reviewer` | Test coverage quality (Council of 5) |
| `ai-team-security-ops-reviewer` | Auth guards, data exposure, input validation (Council of 5) |
| `ai-team-council-synthesizer` | Merges Council of 5 findings into phase verdict |
| `ai-team-qc` | Final ship/no-ship recommendation before merge |

## Project

BalanceIt3 is a private two-user household personal finance app. The north star is a trustworthy, explainable `safe to spend` number calculated from near-term cash flow. Exactly two intended users: a technical admin operator and a non-technical household member.

## Implementation Plan

`docs/plans/2026-04-08-balanceit3-full-development.md`

All agents must read this plan before starting implementation work.

## Product Docs

Canonical requirements are in `balanceit3_docs/`:
- `01_product_brief.md` — what the product is and who it's for
- `02_domain_model.md` — all domain entities (canonical)
- `05_functional_requirements.md` — FR-1 through FR-32
- `07_technical_guardrails.md` — what not to do

Do not treat `balanceit3_docs/history/` as canonical.

## Stack

### Frontend
- **Framework**: Astro 5.x
- **UI**: React 19 (islands for interactive components only)
- **Styling**: Tailwind CSS 4 + shadcn/ui
- **Validation**: Zod + React Hook Form
- **Testing**: Vitest + Testing Library + Playwright

### Backend
- **Framework**: FastAPI (Python 3.11+)
- **ORM**: SQLAlchemy 2.0 (async)
- **Migrations**: Alembic
- **Validation**: Pydantic v2
- **Database**: PostgreSQL via Neon (asyncpg)
- **Package manager**: uv
- **Testing**: pytest + httpx + pytest-asyncio

### Auth
- Google OAuth via FastAPI (allowlisted emails only)
- JWT session tokens
- No public signup — ever

### Deploy
- Frontend: Vercel
- Backend: Railway or Render
- Database: Neon (serverless PostgreSQL)

## Project Structure (target)

```
web/                        — Astro frontend
  src/
    pages/                  — Astro pages (routing)
      sign-in.astro
      index.astro           — household dashboard
      admin/                — admin-only pages
    components/             — Astro + React components
    lib/                    — client-side utilities
  astro.config.mjs
  package.json

backend/                    — FastAPI backend
  app/
    api/v1/                 — route handlers (thin)
    domain/                 — ALL business logic here
      forecasting.py
      safe_to_spend.py
      reconciliation.py
      ingestion.py
      alerts.py
      budgeting.py
    models/                 — SQLAlchemy models
    schemas/                — Pydantic schemas
    core/                   — config, auth, db session
  alembic/                  — migrations
  tests/
  pyproject.toml

docs/plans/                 — implementation plans
balanceit3_docs/            — product requirements (read-only)
```

## Commands

### Frontend (`web/`)
| Purpose | Command |
|---------|---------|
| Dev server | `npm run dev` |
| Build | `npm run build` |
| Unit tests | `npx vitest run` |
| E2E tests | `npx playwright test` |
| Type check | `npx tsc --noEmit` |

### Backend (`backend/`)
| Purpose | Command |
|---------|---------|
| Dev server | `uv run uvicorn app.main:app --reload` |
| Tests | `uv run pytest` |
| Type check | `uv run mypy app/` |
| New migration | `uv run alembic revision --autogenerate -m "description"` |
| Apply migrations | `uv run alembic upgrade head` |

## Architecture Rules

- All business logic belongs in `backend/app/domain/`. Never in route handlers or UI.
- Route handlers: validate input (Pydantic) → call domain function → return response.
- Astro pages handle data fetching server-side. React components receive data as props.
- Immutable source records (Transactions) are never mutated after ingestion.
- Low-confidence assumptions must reduce trust signaling explicitly — never silently degrade.
- Admin overrides must create AuditEvent records.

## Testing Rules

- Domain logic requires tests before implementation (TDD).
- Deepest coverage on `forecasting.py` and `safe_to_spend.py`.
- Both admin and member personas covered in integration/e2e tests.
- Auth guard tests required: unauthenticated redirect, member cannot access admin routes.
- Do not mock the database in integration tests.

## Personas & Roles

| Role | Access |
|------|--------|
| `admin` | All routes including `/admin/*` |
| `member` | Dashboard and read-only views only |

Auth: Google OAuth only. Allowlist controlled by `ALLOWLISTED_EMAILS` env var.

## Git Workflow

- Branch naming: `feat/<topic>`, `fix/<topic>`, `chore/<topic>`, `docs/<topic>`
- Conventional commits required
- Commit after each passing test suite

## Approval Gates

Log to `.ai-team/outstanding-questions.md`. Continue safe work; hold gated actions.

Gated (require approval before proceeding):
- Merging to `main`
- Deploying to production
- Destructive database migrations
- Changing auth config or allowlist

## Priority Rule

FR-1 through FR-6 (forecasting + safe-to-spend) ship first. Nothing else matters until the core number works and is trustworthy.
