# GitHub Issue Draft: Add Validated Transaction Import Workflow

## Summary

Implement CSV import and review behavior that preserves lineage and protects the forecast from duplicate or malformed source data.

## Source Inputs

- feature dossier: `features/04_data_ingestion.md`
- FR references: `FR-7`, `FR-8`, `FR-9`, `FR-10`

## Tasks

- [ ] validate CSV uploads
- [ ] preserve source references on imported transactions
- [ ] detect duplicate candidates before import completion

## Required Tests

- CSV validation cases
- lineage persistence
- duplicate candidate handling
