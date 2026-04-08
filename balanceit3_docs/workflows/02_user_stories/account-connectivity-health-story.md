# User Story: Maintain Healthy Account Connectivity

## Story

As the technical household operator,
I want to see account connection health and control whether an account is used in the forecast,
So that stale or broken sources do not silently distort `safe to spend`.

## Source Inputs

- feature dossier: `features/03_account_connectivity.md`
- FR references: `FR-7`, `FR-11`, `FR-30`

## Acceptance Notes

- sync state is visible
- forecast inclusion can be changed intentionally
- unhealthy accounts surface clear warnings

## Required Tests

- account connection status rendering
- account inclusion toggle behavior
- stale-sync warning behavior
