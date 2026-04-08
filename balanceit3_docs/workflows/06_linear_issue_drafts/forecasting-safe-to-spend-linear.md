# Linear Issue Draft: Forecasting And Safe-To-Spend MVP

## Team

BalanceIt3 Core

## Description

Build the core forecasting and safe-to-spend loop so the dashboard can answer the daily household spending question with explanation and warning states.

## Customer Value

This enables the non-technical household user to decide whether spending is safe without spreadsheet work or admin help.

## Source Inputs

- `features/01_cash_flow_forecasting.md`
- `features/02_safe_to_spend_calculation.md`
- `05_functional_requirements.md`

## Acceptance Criteria

- current `safe to spend` is visible on dashboard
- forecast explanation is available
- low-confidence conditions are surfaced clearly

## Test Outline

- unit coverage for calculation rules
- integration coverage for data assembly
- acceptance coverage for dashboard presentation and warning copy
