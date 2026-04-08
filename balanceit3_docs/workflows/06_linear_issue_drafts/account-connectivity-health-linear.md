# Linear Issue Draft: Account Connectivity Health

## Team

BalanceIt3 Core

## Description

Add account health and forecast participation controls so broken or stale connections do not silently affect spend guidance.

## Customer Value

Improves trust in forecast results by making source freshness visible and controllable.

## Source Inputs

- `features/03_account_connectivity.md`
- `05_functional_requirements.md`

## Acceptance Criteria

- account sync health is visible
- account forecast inclusion is controllable
- stale accounts degrade trust state clearly

## Test Outline

- account health rendering
- inclusion toggle behavior
- stale-state warning behavior
