# User Story: Review Reconciliation Decisions

## Story

As the technical household operator,
I want to review uncertain matches and override them when necessary,
So that the forecast reflects observed reality instead of stale assumptions.

## Source Inputs

- feature dossier: `features/07_reconciliation.md`
- FR references: `FR-16`, `FR-17`, `FR-18`, `FR-19`

## Acceptance Notes

- review queue shows confidence and reason codes
- overrides are explicit and auditable
- forecast confidence updates after review changes

## Required Tests

- match-state transitions
- override behavior
- recalculation after review
