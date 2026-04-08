# User Story: Manage Recurring Obligations

## Story

As the technical household operator,
I want to maintain recurring obligations and their due-state,
So that safe-to-spend calculations protect the bills the household must actually pay.

## Source Inputs

- feature dossier: `features/05_recurring_obligations.md`
- FR references: `FR-13`, `FR-14`, `FR-15`

## Acceptance Notes

- cadence and due date are editable
- obligations can be deactivated or skipped intentionally
- late or missing state is visible to reconciliation and forecasting

## Required Tests

- cadence editing
- due-state transitions
- forecast impact visibility
