# BalanceIt3 Domain Model

## Core Entities

```json
HouseholdUser {
  id: "string",
  role: "admin | member",
  auth_provider: "google",
  email: "string",
  is_allowlisted: true,
  status: "active | disabled"
}
```

```json
Account {
  id: "string",
  provider_type: "manual | connected",
  display_name: "string",
  account_type: "checking | savings | credit | other",
  current_balance: "decimal",
  available_balance: "decimal | null",
  last_synced_at: "datetime | null",
  include_in_forecast: true
}
```

```json
Transaction {
  id: "string",
  account_id: "string",
  posted_at: "date",
  amount: "decimal",
  direction: "inflow | outflow",
  description: "string",
  category_id: "string | null",
  source_type: "csv | connected",
  source_reference: "string",
  is_read_only_source: true
}
```

```json
IncomeSchedule {
  id: "string",
  label: "string",
  cadence: "weekly | biweekly | semimonthly | monthly | custom",
  expected_amount: "decimal",
  next_expected_date: "date",
  confidence_level: "high | medium | low",
  is_active: true
}
```

```json
RecurringObligation {
  id: "string",
  label: "string",
  cadence: "weekly | monthly | annual | custom",
  expected_amount: "decimal",
  next_due_date: "date",
  category_id: "string | null",
  confidence_level: "high | medium | low",
  is_active: true
}
```

```json
ForecastWindow {
  id: "string",
  starts_on: "date",
  ends_on: "date",
  base_balance: "decimal",
  projected_inflows: "decimal",
  projected_outflows: "decimal",
  projected_low_point: "decimal",
  generated_at: "datetime"
}
```

```json
SafeToSpendResult {
  id: "string",
  forecast_window_id: "string",
  amount: "decimal",
  method_version: "string",
  confidence_level: "high | medium | low",
  explanation_summary: "string",
  calculated_at: "datetime"
}
```

```json
ReconciliationRecord {
  id: "string",
  subject_type: "income | obligation | transaction",
  subject_id: "string",
  matched_transaction_id: "string | null",
  status: "matched | review | missing | ignored",
  confidence_score: "decimal",
  reason_code: "string"
}
```

```json
Alert {
  id: "string",
  alert_type: "cash_flow_risk | missing_income | missing_obligation | auth | sync | anomaly",
  severity: "info | warning | critical",
  message: "string",
  related_entity_type: "string | null",
  related_entity_id: "string | null",
  is_resolved: false
}
```

```json
AuditEvent {
  id: "string",
  actor_user_id: "string | system",
  event_type: "auth | ingestion | forecast | override | configuration",
  entity_type: "string",
  entity_id: "string",
  summary: "string",
  occurred_at: "datetime"
}
```

## Relationships

- `HouseholdUser` authenticates and views forecast outputs.
- `Account` owns many `Transaction` records.
- `Transaction` contributes to reconciliation, budgets, and forecasts.
- `IncomeSchedule` and `RecurringObligation` supply expected future cash-flow events.
- `ForecastWindow` aggregates balances, income, and obligations over a selected horizon.
- `SafeToSpendResult` is derived from a `ForecastWindow`.
- `ReconciliationRecord` links actual transactions to expected income or obligations.
- `Alert` highlights conditions that may reduce confidence in the forecast.
- `AuditEvent` records important automated or manual actions.

## Domain Invariants

- public signup is not allowed
- only allowlisted Google identities may authenticate
- source transactions remain traceable to their ingestion origin
- safe-to-spend results must be explainable from forecast inputs
- low-confidence assumptions must reduce trust signaling, not silently disappear
- admin overrides must be auditable

## Derived Concepts

- `forecast confidence`: a rollup of data freshness, reconciliation quality, and schedule confidence
- `household runway`: days until projected low point or negative balance under current assumptions
- `discretionary buffer`: spendable margin after protected obligations and risk buffer are applied

## Glossary

- `safe to spend`: the discretionary amount that can be spent without violating the forecast safety buffer inside the active forecast window
- `protected obligation`: a recurring or expected outflow treated as required for safe-to-spend calculations
- `forecast window`: the near-term period used to compute spendable amount and risk
- `review state`: a condition where the admin user should inspect a match, assumption, or anomaly before trusting the forecast
