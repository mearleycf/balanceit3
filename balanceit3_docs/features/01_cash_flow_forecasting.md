# Cash-Flow Forecasting

## Purpose

Cash-flow forecasting is the core engine that projects near-term household liquidity and determines whether discretionary spending is safe. This feature is upstream of the final `safe to spend` output.

## Why It Matters

- converts balances and schedules into a future-looking financial picture
- identifies low points before they become real account problems
- provides the base material for `safe to spend`

## Inputs

- account balances and account inclusion settings
- recent transactions
- income schedules
- recurring obligations
- reconciliation state
- budget constraints when explicitly enabled

## Core Capabilities

- select an active forecast window
- calculate projected inflows and outflows
- model timing of expected events
- calculate projected low point and runway
- score confidence from source freshness and reconciliation quality
- support explanation of the final output

## Key Requirements

- `FR-1` through `FR-6`
- `FR-16` through `FR-19`
- `FR-30` through `FR-32`

## Primary UX

- dashboard summary
- forecast details page
- confidence and warning explanations
- admin diagnostics for recalculation drivers

## Important Edge Cases

- missing expected income
- duplicate imported transactions
- delayed recurring obligations
- stale account data
- conflicting manual overrides

## Success Signals

- projected low point aligns with known future obligations and income
- admin user can explain why the forecast changed
- non-technical user can understand whether the forecast is trustworthy today
