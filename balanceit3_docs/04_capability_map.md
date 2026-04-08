# Capability Map

## North Star

Safe To Spend
  - Calculate discretionary spendable amount
  - Explain drivers of the current amount
  - Show confidence and warnings
  - Recompute when source conditions change

## Forecasting Foundation

Cash-Flow Forecasting
  - Build forecast windows
  - Project income events
  - Project recurring obligations
  - Model low-point and runway
  - Support confidence and buffer logic

## Data Foundation

Account Connectivity
  - Connect supported financial accounts
  - Refresh balances and transaction feeds
  - Surface sync health

Data Ingestion
  - Import CSV transactions
  - Normalize source records
  - Deduplicate imported activity
  - Preserve lineage and source references

## Expected Event Modeling

Income
  - Create income schedules
  - Detect expected deposit timing
  - Track expected versus actual income

Recurring Obligations
  - Create required outflow schedules
  - Support common cadence types
  - Track due, late, skipped, and changed obligations

## Validation And Trust

Reconciliation
  - Match actual transactions to expected events
  - Flag missing or late items
  - Support manual review and overrides

Alerts
  - Warn about forecast risk
  - Warn about sync or auth problems
  - Warn about low-confidence assumptions

Trust
  - Explain the current forecast state
  - Expose confidence and data freshness
  - Show why a result changed

Auditability
  - Record key auth and override events
  - Preserve explanation trail for calculations
  - Trace automated decisions to source inputs

## Household Guidance

Budgeting
  - Define discretionary category limits
  - Compare spend versus plan
  - Feed constraints back into safe spending guidance

## Supporting Experience

Household UX
  - Surface the key answer immediately
  - Separate admin actions from daily-use flows
  - Keep important detail available without overload
