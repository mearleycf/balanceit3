# BalanceIt3 Agent Guide

## Project

BalanceIt3 is a private two-user household personal finance app. The north star is a trustworthy, explainable `safe to spend` number calculated from near-term cash flow. It is not a public SaaS product. It has exactly two intended users: a technical admin operator and a non-technical household member.

## How To Start Work

To begin building this project, invoke the orchestrator:

```
/ai-team-orchestrator
```

The orchestrator reads this file and the implementation plan, then dispatches the right specialist agents per task. You do not need to invoke specialist agents directly — the orchestrator does that.

## Implementation Plan

The full step-by-step implementation plan is at:
`docs/plans/2026-04-08-balanceit3-full-development.md`

All agents should read this plan before starting any implementation work.

## Agent Roles

| Agent | Responsibility |
|-------|---------------|
| `ai-team-orchestrator` | **Start here.** Reads the plan, dispatches specialist agents, runs Council of 5 at phase boundaries, manages approval gates |
| `ai-team-backend-developer` | Domain logic (`src/lib/domain/`), API routes, Prisma schema, auth config |
| `ai-team-frontend-developer` | UI pages, components, client-side interactions |
| `ai-team-devops` | Vercel deployment, env config, CI |
| `ai-team-dev-deploy-specialist` | Deploy artifact preparation, rollout guidance |
| `ai-team-qa` | Test strategy, test suite design |
| `ai-team-ux-designer` | UX flow, component design decisions |
| `ai-team-security-ops-reviewer` | Auth guards, data exposure, input validation |
| `ai-team-product-acceptance-reviewer` | FR coverage validation |
| `ai-team-architecture-reviewer` | Layer boundary enforcement, domain separation |
| `ai-team-code-quality-reviewer` | Code clarity, style, maintainability |
| `ai-team-test-reliability-reviewer` | Test coverage quality |
| `ai-team-council-synthesizer` | Merges Council of 5 findings into phase verdict |
| `ai-team-qc` | Final ship/no-ship recommendation before merge |

## Product Docs

Canonical product requirements are in `balanceit3_docs/`. Key files:
- `balanceit3_docs/01_product_brief.md` — what the product is and who it's for
- `balanceit3_docs/02_domain_model.md` — all domain entities (canonical)
- `balanceit3_docs/05_functional_requirements.md` — FR-1 through FR-32
- `balanceit3_docs/07_technical_guardrails.md` — what not to do

Do not treat `balanceit3_docs/history/` as canonical requirements.

## Stack

- **Framework**: Next.js 15 (App Router, TypeScript)
- **Database**: PostgreSQL via Neon (serverless managed)
- **ORM**: Prisma
- **Auth**: NextAuth v5 with Google OAuth — allowlisted emails only, no public signup
- **UI**: React 19, Tailwind CSS, shadcn/ui
- **Validation**: Zod
- **Unit/Integration Tests**: Vitest + Testing Library
- **E2E Tests**: Playwright
- **Deploy**: Vercel

## Project Structure (target)

```
src/
  app/
    (auth)/sign-in/      — sign-in page
    (dashboard)/         — household daily-use view
    (admin)/             — admin-only routes, role-guarded
    api/                 — API routes
  lib/
    domain/              — ALL business logic lives here (forecasting, safe-to-spend, reconciliation, ingestion, alerts, budgeting)
    auth.ts              — NextAuth config
    auth-utils.ts        — allowlist helpers
    prisma.ts            — Prisma client singleton
  components/            — shared UI components
  test/setup.ts          — Vitest global setup
prisma/
  schema.prisma
docs/
  plans/                 — implementation plans
balanceit3_docs/         — product requirements (read-only reference)
```

## Commands

| Purpose | Command |
|---------|---------|
| Dev server | `npm run dev` |
| Unit tests | `npx vitest run` |
| Unit tests (watch) | `npx vitest` |
| E2E tests | `npx playwright test` |
| Type check | `npx tsc --noEmit` |
| Lint | `npx eslint src/ --ext .ts,.tsx` |
| DB migrate (dev) | `npx prisma migrate dev` |
| DB migrate (prod) | `npx prisma migrate deploy` |
| DB studio | `npx prisma studio` |
| Generate Prisma client | `npx prisma generate` |

## Testing Rules

- Domain logic in `src/lib/domain/` requires unit tests before implementation (TDD).
- Deepest test coverage required for `forecasting.ts` and `safe-to-spend.ts` — these are the north star.
- API routes require at least happy-path and auth-guard tests.
- UI components require interaction tests via Testing Library.
- E2E tests cover: dashboard visible to household user, admin routes blocked for member role, unauthenticated redirect.
- Do not mock the database in integration tests — use Prisma with a test database.

## Architecture Rules

- All business logic belongs in `src/lib/domain/`. Never in API route handlers or UI components.
- API routes are thin — validate input with Zod, call a domain function, return the result.
- Separate immutable source records (Transactions) from derived outputs (ForecastWindow, SafeToSpendResult).
- Low-confidence assumptions must reduce trust signaling explicitly — never silently degrade.
- Admin overrides must create AuditEvent records.

## Personas & Roles

| Role | Access |
|------|--------|
| `ADMIN` | All routes including `/admin/*` |
| `MEMBER` | Dashboard and read-only views only |

Auth is Google OAuth only. Allowlist is controlled by `ALLOWLISTED_EMAILS` env var. Public registration is rejected unconditionally.

## Git Workflow

- Branch naming: `feat/<topic>`, `fix/<topic>`, `chore/<topic>`, `docs/<topic>`
- Conventional commits required
- Do not force-push shared history
- Commit after each passing test suite (TDD rhythm)

## Approval Gates

Log approval needs to `.ai-team/outstanding-questions.md`. Continue safe work and hold only gated finalization actions.

Gated actions requiring approval before proceeding:
- Merging to `main`
- Deploying to Vercel production
- Deleting database records or running destructive migrations
- Changing auth configuration or allowlist
- Any action that affects the other household user's experience

## Priority Rule

The `safe to spend` number is the only thing that matters at launch. Do not let decorative UI, broad feature coverage, or secondary concerns delay forecasting accuracy and explainability. FR-1 through FR-6 ship first.
