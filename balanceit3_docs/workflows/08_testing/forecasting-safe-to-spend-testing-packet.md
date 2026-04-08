# Forecasting And Safe-To-Spend Testing Packet

## Source Inputs

- `features/01_cash_flow_forecasting.md`
- `features/02_safe_to_spend_calculation.md`
- `05_functional_requirements.md`
- `06_non_functional_requirements.md`

## Critical Suites

### Unit Testing

- verify forecast window math
- verify projected low-point calculation
- verify safety buffer and confidence adjustments
- verify safe-to-spend amount never ignores protected obligations

### Integration Testing

- verify balances, transactions, income schedules, and obligations assemble into a coherent forecast
- verify reconciliation state changes update forecast confidence
- verify dashboard data payload contains amount, explanation, and warnings

### End-To-End And Functional Testing

- verify non-technical user can sign in and immediately see `safe to spend`
- verify warning state appears when expected events are missing or stale
- verify admin user can correct reconciliation state and see updated results

### API And Contract Testing

- verify forecast endpoint contracts for amount, confidence, explanation summary, and warnings
- verify stale-state and low-confidence flags are explicitly represented

### Security And Auth Testing

- verify only allowlisted Google identities can sign in
- verify unapproved identities are rejected
- verify admin-only diagnostic paths are not exposed to the non-admin user

### Acceptance And Regression Testing

- verify the app still answers the core spending question after changes to ingestion, schedules, or alerts
- verify explanation quality does not regress when forecast logic changes

## Required Now

- unit
- integration
- one end-to-end household flow
- auth allowlist validation
- acceptance coverage for warning states

## Future Hardening

- broader performance/load testing
- long-running stale-data recovery checks
- richer cross-device matrix

## Edge Cases

- paycheck arrives early
- large expected bill is missing
- duplicated import inflates balance confidence
- stale sync state with otherwise unchanged balances
- admin override conflicts with automated reconciliation
