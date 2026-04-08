# GitHub Issue Draft: Add Account Connectivity Health And Forecast Controls

## Summary

Implement account connectivity management so the admin user can understand sync health and control whether an account participates in forecast calculations.

## Source Inputs

- feature dossier: `features/03_account_connectivity.md`
- FR references: `FR-7`, `FR-11`, `FR-30`

## Tasks

- [ ] show connection health and last sync state
- [ ] allow account inclusion or exclusion from forecast logic
- [ ] surface stale or broken connection warnings

## Required Tests

- sync-health rendering
- inclusion toggle behavior
- stale connection warnings
