# Linear Issue Draft: Validated Transaction Import

## Team

BalanceIt3 Core

## Description

Add import workflows that validate CSVs, preserve lineage, and detect duplicate activity before it pollutes the forecast.

## Customer Value

Improves correctness of the data feeding the forecast and `safe to spend`.

## Source Inputs

- `features/04_data_ingestion.md`
- `05_functional_requirements.md`

## Acceptance Criteria

- invalid files return actionable feedback
- imported records preserve lineage
- duplicate candidates are visible before acceptance

## Test Outline

- CSV validation
- lineage persistence
- duplicate detection behavior
