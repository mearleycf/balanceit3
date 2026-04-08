# BalanceIt2 Source Map

## Purpose

This file records which BalanceIt2 sources informed the BalanceIt3 workspace.

## Primary Inputs

- root requirements:
  - `/Users/mikeearley/code/balanceit2/requirements.md`
- feature specs:
  - `/Users/mikeearley/code/balanceit2/project_specs/transaction-data-ingestion/spec.md`
  - `/Users/mikeearley/code/balanceit2/project_specs/recurring-transaction-management/spec.md`
  - `/Users/mikeearley/code/balanceit2/project_specs/transaction-reconciliation/spec.md`
  - `/Users/mikeearley/code/balanceit2/project_specs/budget-management/spec.md`
  - `/Users/mikeearley/code/balanceit2/project_specs/income-forecasting/spec.md`
  - `/Users/mikeearley/code/balanceit2/project_specs/business-expense-tracking/spec.md`
  - `/Users/mikeearley/code/balanceit2/project_specs/pwa-deployment-auth/spec.md`
- architecture context:
  - `/Users/mikeearley/code/balanceit2/backend_v2/docs/architecture/README.md`
- roadmap context:
  - `/Users/mikeearley/code/balanceit2/docs/roadmap/current.md`
- data model hints:
  - `/Users/mikeearley/code/balanceit2/backend_v2/models/transaction.py`
  - `/Users/mikeearley/code/balanceit2/backend_v2/models/recurring.py`
  - `/Users/mikeearley/code/balanceit2/backend_v2/models/reconciliation.py`
  - `/Users/mikeearley/code/balanceit2/backend_v2/models/budget_management.py`

## How Sources Were Used

- product scope and legacy priorities informed the BalanceIt3 capability map
- current data-model concepts informed the successor domain vocabulary
- existing auth and PWA notes informed the private-hosting and allowlisted-Google-auth model
- roadmap emphasis on Safe to Spend informed BalanceIt3's north-star positioning

## What Was Not Carried Forward As A Constraint

- a one-to-one commitment to the current BalanceIt2 implementation stack
- public or multi-tenant growth assumptions
- enterprise-grade compliance or accessibility posture
