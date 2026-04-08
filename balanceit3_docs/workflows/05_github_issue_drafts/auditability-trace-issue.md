# GitHub Issue Draft: Add Auditability And Trace View

## Summary

Implement a household-grade audit and trace view so the admin user can understand auth, config, override, and forecast-affecting events.

## Source Inputs

- feature dossier: `features/11_auditability.md`
- FR references: `FR-9`, `FR-28`, `FR-31`, `FR-32`

## Tasks

- [ ] capture important events consistently
- [ ] surface events in an inspectable audit view
- [ ] connect events back to source lineage and forecast state

## Required Tests

- event capture coverage
- audit view rendering
- traceability behavior
