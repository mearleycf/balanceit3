# BalanceIt3 Product Brief

## Problem Statement

Household cash-flow decisions are hard when account balances, scheduled obligations, expected income, and recently imported transactions are all visible in different places or carry different levels of confidence. A raw bank balance does not answer the question, "What can we safely spend before the next income event?"

BalanceIt3 exists to answer that question with a trustworthy, explainable `safe to spend` number.

## Target Users

- primary operator: a highly technical household owner who designs, develops, administers, and tunes the system
- primary daily consumer: a low-technical-comfort household user who needs clear balances, recent transactions, near-term forecasts, and safe spending guidance

## Core Value Proposition

- convert raw financial data into a reliable, understandable short-horizon cash-flow forecast
- surface a single `safe to spend` amount that accounts for expected income, recurring obligations, recent activity, and forecast risk
- provide enough explanation and review tooling that the result can be trusted and corrected when needed

## Product Priorities

1. Cash-flow forecasting accuracy
2. Safe-to-spend clarity and trustworthiness
3. Low-friction household usage
4. Admin-level explainability and operability
5. Private, secure, low-maintenance deployment

## Key Features

- account connectivity for current balance visibility
- transaction ingestion from CSV and connected accounts
- recurring obligation tracking
- income schedule tracking
- reconciliation between expected and actual events
- safe-to-spend calculation with explainability
- budget guidance for discretionary categories
- alerts for risk, drift, and unusual conditions
- auditability for major automated and manual decisions

## Success Metrics

- daily `safe to spend` amount is available and understandable within seconds of login
- near-term forecast correctly accounts for all known recurring obligations and income events
- household users can identify recent transactions and projected cash-flow risks without admin help
- admin user can trace a forecast or safe-to-spend result back to underlying assumptions and source records
- authentication remains limited to the intended two users with zero public registrations

## Constraints

### Product Constraints

- private household scope, not a public multi-tenant SaaS product
- only two intended authenticated users
- safe-to-spend confidence matters more than broad feature count

### Technical Constraints

- deployment should fit lightweight hosted environments such as Vercel or Hostinger
- the system should prefer managed or low-ops components where possible
- docs must remain portable and interpretable by both humans and AI agents

### Operational Constraints

- the technical household user is the administrator and support path
- the non-technical household user should rarely need to interpret technical state
- the product should tolerate gradual iteration rather than a big-bang launch
