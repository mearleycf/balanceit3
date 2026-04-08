# Functional Requirements

## Forecasting And Safe To Spend

- `FR-1`: The system must calculate a current `safe to spend` amount from account balances, expected income, recurring obligations, and forecast safety rules.
- `FR-2`: The system must present the current `safe to spend` amount as the primary household dashboard outcome.
- `FR-3`: The system must explain the major inputs and assumptions that produced the current `safe to spend` amount.
- `FR-4`: The system must recalculate forecast outputs when balances, transactions, income schedules, obligations, or overrides change.
- `FR-5`: The system must surface a forecast confidence level derived from data freshness, reconciliation quality, and assumption certainty.
- `FR-6`: The system must show projected low point and near-term cash-flow risk inside the active forecast window.

## Accounts And Ingestion

- `FR-7`: The system must allow the admin user to add household accounts through supported connectivity flows or manual import workflows.
- `FR-8`: The system must import CSV transaction files with validation feedback when source formatting is incomplete or ambiguous.
- `FR-9`: The system must preserve source references and lineage for imported or synced transactions.
- `FR-10`: The system must prevent accidental duplication of the same transaction when the same source activity is imported or synced multiple times.
- `FR-11`: The system must allow the admin user to include or exclude specific accounts from forecast calculations.

## Income And Obligations

- `FR-12`: The system must allow the admin user to define expected income schedules with cadence, expected amount, and next date.
- `FR-13`: The system must allow the admin user to define recurring obligations with cadence, expected amount, and next due date.
- `FR-14`: The system must support active and inactive states for income schedules and recurring obligations.
- `FR-15`: The system must represent skipped, late, or changed expected events without deleting prior history.

## Reconciliation And Review

- `FR-16`: The system must compare actual transactions to expected income and recurring obligations.
- `FR-17`: The system must classify reconciliation outcomes into matched, review, missing, or ignored states.
- `FR-18`: The system must allow the admin user to accept, reject, or override reconciliation outcomes.
- `FR-19`: The system must update forecast confidence and explanations when key reconciliation states change.

## Household Experience

- `FR-20`: The system must let the non-technical household user view current `safe to spend`, recent transactions, and near-term forecast highlights without entering admin flows.
- `FR-21`: The system must present plain-language warnings for low confidence, missing expected events, or cash-flow risk.
- `FR-22`: The system must separate admin-only setup and maintenance interfaces from daily-use household views.

## Budgeting And Guidance

- `FR-23`: The system must allow the admin user to define discretionary budget targets for selected categories or spending groups.
- `FR-24`: The system must compare actual discretionary spending against budget targets.
- `FR-25`: The system may use budget state as an input into safe-to-spend guidance, but must keep the calculation explainable.

## Security, Audit, And Access

- `FR-26`: The system must allow authentication only for allowlisted Google identities.
- `FR-27`: The system must reject public self-registration and unapproved new users.
- `FR-28`: The system must record important auth, configuration, override, and forecast-generation events in an audit trail.
- `FR-29`: The system must keep admin controls available only to the technical household operator role.

## Operational Requirements

- `FR-30`: The system must surface sync failures, stale data, or broken assumptions that may materially affect forecast outputs.
- `FR-31`: The system must provide enough operational context for the admin user to diagnose why a result is wrong.
- `FR-32`: The system must retain historical forecast results or explanation summaries long enough to compare output changes over time.
