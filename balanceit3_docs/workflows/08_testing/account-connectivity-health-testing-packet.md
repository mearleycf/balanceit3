# Account Connectivity Health Testing Packet

## Source Inputs

- `features/03_account_connectivity.md`
- `05_functional_requirements.md`

## Critical Suites

- unit: sync health state mapping and inclusion logic
- integration: account status to forecast trust behavior
- API or contract: account health payload shape
- acceptance: stale connection warning and inclusion control behavior
- security: auth and admin-only access to connectivity controls

## Required Now

- account status rendering
- inclusion toggle handling
- stale-state warning behavior

## Future Hardening

- provider-specific failure simulation
- larger cross-device settings coverage
