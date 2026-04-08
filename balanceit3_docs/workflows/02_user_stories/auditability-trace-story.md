# User Story: Trace Important System Decisions

## Story

As the technical household operator,
I want to inspect important auth, config, and forecast-related events,
So that I can understand why the system reached its current state.

## Source Inputs

- feature dossier: `features/11_auditability.md`
- FR references: `FR-9`, `FR-28`, `FR-31`, `FR-32`

## Acceptance Notes

- important events are visible
- overrides and auth events are traceable
- source lineage remains inspectable

## Required Tests

- event capture coverage
- event display behavior
- lineage trace visibility
