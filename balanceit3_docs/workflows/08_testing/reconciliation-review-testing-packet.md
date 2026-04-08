# Reconciliation Review Testing Packet

## Source Inputs

- `features/07_reconciliation.md`
- `05_functional_requirements.md`

## Critical Suites

- unit: match-state and confidence transitions
- integration: review action updates forecast trust and explanation
- acceptance: queue review behavior for admin user
- regression: prior good matches remain stable after queue changes

## Required Now

- queue behavior
- override behavior
- recalculation after review

## Future Hardening

- large queue handling
- more nuanced confidence-threshold tuning
