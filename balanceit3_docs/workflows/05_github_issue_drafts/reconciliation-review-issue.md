# GitHub Issue Draft: Add Reconciliation Review Queue

## Summary

Implement reconciliation review behavior so the admin user can inspect uncertain matches and correct forecast assumptions safely.

## Source Inputs

- feature dossier: `features/07_reconciliation.md`
- FR references: `FR-16`, `FR-17`, `FR-18`, `FR-19`

## Tasks

- [ ] show review queue with confidence and reason codes
- [ ] allow accept, reject, ignore, and override actions
- [ ] update forecast confidence after review changes

## Required Tests

- queue filtering
- action-state transitions
- recalculation behavior
