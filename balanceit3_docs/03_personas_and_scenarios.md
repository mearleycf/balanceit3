# Personas And Scenarios

## Persona 1: Technical Household Operator

- designs and develops the application
- configures data sources, rules, and deployment
- serves as the only expected administrator in production
- wants explainability, override controls, audit history, and clear operational state
- tolerates complexity when it increases forecast quality or system trust

### Core Goals

- make the `safe to spend` amount correct and explainable
- identify gaps in ingestion, reconciliation, or forecasting assumptions
- keep the system secure and low-maintenance
- inspect and tune edge cases without corrupting source data

### Constraints

- limited time for ongoing operations
- wants a lightweight hosting and auth model
- needs enough observability to diagnose wrong outputs quickly

## Persona 2: Non-Technical Household User

- has very limited technology comfort
- is not an administrator and should not need to act like one
- primarily checks balances, recent transactions, upcoming income and obligations, and `safe to spend`
- values clarity, confidence, and ease over configurability

### Core Goals

- know whether money can be spent safely
- see the most recent important transactions
- understand major upcoming bills or income events
- recognize when a warning matters and when it does not

### Constraints

- should not need to interpret technical jargon
- should not be asked to maintain categories, rules, or integrations
- should not be forced through complex workflows to get the key answer

## Scenario 1: Morning Safe-To-Spend Check

1. Non-technical household user signs in with Google.
2. The dashboard opens directly to the current `safe to spend` amount.
3. A supporting summary explains:
   - current account position
   - next expected income event
   - next major obligations
   - any warning affecting confidence
4. The user reviews recent transactions and decides whether discretionary spending is safe today.

## Scenario 2: Forecast Drift After A Missing Charge

1. Technical household operator notices the forecast confidence dropped.
2. The app highlights an expected recurring obligation with no matching transaction.
3. The operator reviews reconciliation state and either confirms the obligation is late, marks it as skipped, or adjusts the schedule.
4. The forecast and `safe to spend` result recalculate with an updated explanation trail.

## Scenario 3: Paycheck Lands Earlier Than Expected

1. A connected or imported transaction matches an expected income event.
2. Reconciliation updates the income schedule.
3. The forecast window recalculates and increases the `safe to spend` amount if appropriate.
4. Both users can see the updated cash-flow picture without manual spreadsheet work.

## Scenario 4: New Account Or Import Source Added

1. Technical household operator adds a new connected account or imports a CSV.
2. The system validates source data, deduplicates where possible, and marks uncertain items for review.
3. Transactions become eligible for balance display, reconciliation, and forecasting.
4. The non-technical user benefits from more complete household coverage without interacting with setup flows.

## Scenario 5: Household User Sees A Warning

1. Non-technical household user sees a yellow warning next to `safe to spend`.
2. The warning uses plain language such as "One expected bill has not matched yet."
3. The user understands that the number may be conservative or uncertain.
4. If action is required, the interface directs the issue to the technical operator rather than forcing the user through admin flows.
