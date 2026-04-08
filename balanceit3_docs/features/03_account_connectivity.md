# Account Connectivity

## Purpose

Account connectivity brings current balances and transaction feeds into the product with less manual effort.

## Core Capabilities

- connect supported household accounts
- refresh balances and transactions
- surface sync health and stale-state warnings
- allow account inclusion or exclusion from forecast calculations

## Key Requirements

- `FR-7`
- `FR-11`
- `FR-30`

## Constraints

- optimized for a small number of household accounts
- must not weaken the private two-user auth model
- should degrade gracefully when sync providers are unavailable
