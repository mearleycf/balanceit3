# GitHub Issue Draft: Build Forecasting And Safe-To-Spend Dashboard Loop

## Summary

Implement the core household dashboard loop that calculates and presents a trustworthy `safe to spend` amount backed by near-term cash-flow forecasting.

## Source Inputs

- feature dossiers:
  - `features/01_cash_flow_forecasting.md`
  - `features/02_safe_to_spend_calculation.md`
- FR references:
  - `FR-1` through `FR-6`
  - `FR-20`, `FR-21`, `FR-31`, `FR-32`

## Tasks

- [ ] model forecast window and safe-to-spend result
- [ ] calculate projected inflows, outflows, and low point
- [ ] derive confidence-aware safe-to-spend amount
- [ ] present amount, explanation, and warnings on dashboard
- [ ] add grouped tests for critical behavior

## Required Tests

- unit tests for forecast math and buffer rules
- integration tests for dashboard data assembly
- acceptance coverage for low-confidence warnings
