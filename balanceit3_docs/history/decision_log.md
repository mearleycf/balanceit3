# Decision Log

## D-001: BalanceIt3 Is A Successor Workspace

- decision: treat this package as the source of truth for a successor product rather than a rewrite checklist for the current app
- reason: the goal is to create a portable documentation workspace that can seed a new project or docs branch

## D-002: Safe To Spend Is The Product North Star

- decision: prioritize `cash-flow forecasting` and `safe-to-spend calculation` above all other feature areas
- reason: this is the primary household decision outcome and the clearest product differentiator

## D-003: Two-Persona Model Only

- decision: model only the technical household operator and the non-technical household user
- reason: the app is intentionally scoped to a two-user household deployment

## D-004: Private Auth Model

- decision: require allowlisted Google OAuth and reject public registration
- reason: the app is not intended to support open signup or broader account management

## D-005: Mermaid As Canonical Diagram Format

- decision: use Mermaid in canonical docs for flows and site maps
- reason: Mermaid renders in Obsidian and is easy for AI agents to interpret

## D-006: Workflow Artifacts Included In The Workspace

- decision: include GitHub, Linear, and testing scaffolding inside the package
- reason: the workspace should generate actionable delivery artifacts, not only static source docs
