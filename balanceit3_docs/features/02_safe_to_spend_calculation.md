# Safe-To-Spend Calculation

## Purpose

Safe-to-spend calculation is the single most important feature in BalanceIt3. It turns forecast information into a household decision number that answers, "How much can we spend without causing avoidable near-term trouble?"

## Why It Matters

- it is the primary daily-use answer for the non-technical household user
- it creates the central trust contract of the whole product
- it determines whether the rest of the forecast system is delivering practical value

## Core Capabilities

- derive spendable amount from forecast outputs
- protect known obligations before exposing discretionary buffer
- apply confidence-aware safety rules
- explain the amount in plain language
- update when relevant inputs change

## Inputs

- current included account balances
- projected income and projected obligations
- forecast low point and runway
- confidence level
- optional discretionary budget rules

## Output Shape

- current amount
- confidence label
- short explanation summary
- top drivers increasing or decreasing the amount
- warning state if the number is conservative or uncertain

## Key Requirements

- `FR-1` through `FR-6`
- `FR-20` and `FR-21`
- `FR-31` and `FR-32`

## Edge Cases

- negative or near-zero safe-to-spend result
- large upcoming obligation not yet matched
- unexpectedly early paycheck
- low-confidence schedule assumptions
- stale sync state with a previously healthy forecast

## Testing Priority

This dossier requires the deepest coverage in the workspace. Any testing strategy that under-specifies this feature is incomplete.
