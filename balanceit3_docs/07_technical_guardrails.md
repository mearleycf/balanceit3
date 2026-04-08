# Technical Guardrails

## Architecture Preferences

- prefer a clear separation between UI, domain logic, data access, and integration layers
- treat forecasting and safe-to-spend logic as a first-class domain module, not scattered UI math
- keep the system portable across lightweight hosting environments
- preserve human-readable artifacts and explanations where possible

## Deployment Preferences

- optimize for simple web deployment rather than complex self-hosted infrastructure
- prefer platform capabilities that reduce maintenance burden
- keep database and auth choices compatible with private two-user operation

## Product Guardrails

- never let the product prioritize a decorative dashboard over correct forecast outcomes
- do not expose admin complexity to the non-technical user by default
- do not accept public signup or open registration flows
- do not hide forecast uncertainty behind a single confident-looking number

## Data Guardrails

- keep transaction lineage and source references
- preserve enough history to explain forecast changes
- separate immutable source records from derived planning and forecast outputs

## Testing Guardrails

- give the deepest test coverage to `cash-flow forecasting` and `safe-to-spend calculation`
- require scenario coverage for both personas
- use grouped testing plans that map directly back to functional requirements

## Documentation Guardrails

- top-level numbered docs are canonical
- feature dossiers must stay separate by feature area
- workflow artifacts must cite their source feature or FR set
- diagrams should use Mermaid as the canonical format for portability and AI readability

## Technology Stance

BalanceIt3 is free to evolve beyond BalanceIt2's current stack. This workspace should capture technical preferences and constraints, but it should not lock the successor project to a one-for-one rewrite of the existing architecture unless a later decision log explicitly chooses that path.
