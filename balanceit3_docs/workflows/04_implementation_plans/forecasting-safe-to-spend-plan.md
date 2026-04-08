# Implementation Plan: Forecasting And Safe To Spend

## Goal

Implement the core BalanceIt3 loop that calculates and presents `safe to spend` from a near-term household forecast.

## Source Inputs

- canonical docs:
  - `01_product_brief.md`
  - `02_domain_model.md`
  - `05_functional_requirements.md`
- feature dossiers:
  - `features/01_cash_flow_forecasting.md`
  - `features/02_safe_to_spend_calculation.md`
- acceptance criteria:
  - `workflows/03_acceptance_criteria/safe-to-spend-dashboard-criteria.md`

## Work Slices

1. model forecast window and safe-to-spend result objects
2. compute forecast inputs from balances, income, obligations, and reconciliation state
3. derive explainable safe-to-spend output
4. expose dashboard presentation and warning states
5. validate with grouped testing packet
