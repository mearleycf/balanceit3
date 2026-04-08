# Page Inventory And Actions

This document complements the sitemap in `09_ux_flows.md`. The sitemap shows
where users can navigate. This document explains what each page is for, who
can access it, what they can see, and what they can do there.

## Usage Notes

- `member` means the non-technical household user.
- `admin` means the technical household operator.
- If a page contains both viewing and action behavior, the viewing contract
  should remain useful even when admin-only actions are hidden.

## Primary Actions Summary

| Page | Primary User | Main Things You Can Do |
| --- | --- | --- |
| Sign-In | member, admin | sign in with allowlisted Google account |
| Dashboard | member, admin | check `safe to spend`, review recent activity, follow warnings |
| Forecast Details | member, admin | understand drivers, inspect forecast state, compare changes |
| Transactions | member, admin | browse, search, and filter household activity |
| Alerts | member, admin | review risk and warning conditions, jump to affected areas |
| Admin Overview | admin | choose maintenance and diagnostic workflows |
| Accounts And Connectivity | admin | add accounts, refresh sync, include or exclude accounts |
| CSV Imports | admin | upload files, review validation, accept import results |
| Income Schedules | admin | create and maintain expected income events |
| Recurring Obligations | admin | create and maintain required outgoing events |
| Reconciliation Queue | admin | accept, reject, or override uncertain matches |
| Budget Settings | admin | maintain discretionary targets and guidance inputs |
| Audit Trail | admin | inspect important auth, config, and forecast events |
| User Access | admin | manage allowlisted users and roles |

## 1. Sign-In Page

### Purpose

Authenticate an allowlisted household user with Google and route them into the
correct product experience.

### Access

- anonymous visitor

### Users Can See

- Google sign-in entry point
- clear message that access is limited to approved household users
- rejection or error state if access is denied

### Users Can Do

- start Google OAuth sign-in
- retry after transient auth failure

### Key States

- signed out
- auth in progress
- allowlist rejection
- provider error

### Next Navigation

- `Dashboard`

## 2. Dashboard

### Purpose

Answer the daily household question: "What is safe to spend right now?"

### Access

- `member`
- `admin`

### Users Can See

- current `safe to spend` amount
- confidence indicator
- short explanation summary
- recent transactions
- upcoming income
- upcoming obligations
- high-priority warnings

### Users Can Do

- review whether discretionary spending is safe today
- inspect recent transaction activity
- inspect upcoming expected events
- open deeper forecast detail
- open alerts
- for `admin`, jump into diagnostic or admin workflows

### Key States

- healthy trusted forecast
- conservative forecast with warnings
- stale data state
- missing expected event state
- loading or unavailable state

### Next Navigation

- `Forecast Details`
- `Transactions`
- `Alerts`
- `Admin`

## 3. Forecast Details

### Purpose

Explain how the current forecast and `safe to spend` result were produced.

### Access

- `member`
- `admin`

### Users Can See

- active forecast window
- projected inflows and outflows
- projected low point and runway
- explanation trail
- confidence drivers
- what changed since last meaningful update

### Users Can Do

- understand why the current number is high, low, or uncertain
- inspect major forecast drivers
- compare current state to previous state
- for `admin`, jump into the relevant corrective workflow

### Key States

- stable explanation
- changed-input explanation
- low-confidence explanation

### Next Navigation

- `Dashboard`
- `Transactions`
- `Alerts`
- `Reconciliation Queue` for `admin`

## 4. Transactions

### Purpose

Provide a browsable record of recent household transaction activity and source
lineage.

### Access

- `member`
- `admin`

### Users Can See

- transaction feed
- transaction dates, descriptions, and amounts
- search and filters
- source and lineage metadata when needed

### Users Can Do

- browse recent activity
- search for a known transaction
- filter by account, time period, or direction
- for `admin`, inspect source quality and related reconciliation context

### Key States

- normal activity view
- filtered result set
- missing or delayed import state
- duplicate or suspicious source state for `admin`

### Next Navigation

- `Dashboard`
- `Forecast Details`
- `Alerts`

## 5. Alerts

### Purpose

Surface important conditions that reduce trust in the forecast or require
attention.

### Access

- `member`
- `admin`

### Users Can See

- cash-flow risk warnings
- missing income warnings
- missing obligation warnings
- sync or auth health warnings
- severity and explanation summary

### Users Can Do

- understand whether the current forecast is safe to trust
- drill into the affected area
- for `admin`, route directly to the page where the issue can be resolved

### Key States

- no active alerts
- informational alerts
- warning alerts
- critical alerts

### Next Navigation

- `Dashboard`
- `Forecast Details`
- relevant admin page for `admin`

## 6. Admin Overview

### Purpose

Give the technical household operator one place to access maintenance and
diagnostic workflows.

### Access

- `admin`

### Users Can See

- admin navigation
- system health snapshot
- data freshness indicators
- outstanding review or warning queues

### Users Can Do

- choose an admin workflow
- prioritize maintenance tasks
- diagnose which subsystem is affecting trust

### Next Navigation

- `Accounts And Connectivity`
- `CSV Imports`
- `Income Schedules`
- `Recurring Obligations`
- `Reconciliation Queue`
- `Budget Settings`
- `Audit Trail`
- `User Access`

## 7. Accounts And Connectivity

### Purpose

Manage connected and manually tracked household accounts that feed the forecast.

### Access

- `admin`

### Users Can See

- list of active accounts
- connection status
- last sync timestamp
- account inclusion or exclusion state

### Users Can Do

- add a connected account
- refresh account sync
- disable or remove an account from forecast participation
- inspect sync health

### Key States

- connected and healthy
- stale sync
- sync error
- excluded from forecast

### Next Navigation

- `Admin Overview`
- `CSV Imports`
- `Transactions`

## 8. CSV Imports

### Purpose

Bring transaction data into the system from manual files and make import quality
visible.

### Access

- `admin`

### Users Can See

- upload entry point
- validation summary
- import result summary
- duplicate or ambiguity warnings

### Users Can Do

- upload a CSV file
- review normalization and validation results
- inspect suspected duplicates
- accept a successful import into the forecast data set

### Key States

- awaiting upload
- validation failed
- imported with warnings
- imported successfully

### Next Navigation

- `Admin Overview`
- `Transactions`
- `Reconciliation Queue`

## 9. Income Schedules

### Purpose

Maintain expected income events that the forecast depends on.

### Access

- `admin`

### Users Can See

- existing income schedules
- cadence and expected amount
- next expected date
- confidence or drift indicators

### Users Can Do

- create an income schedule
- edit cadence, amount, or next date
- pause or deactivate a schedule
- inspect match quality against real deposits

### Next Navigation

- `Admin Overview`
- `Reconciliation Queue`
- `Forecast Details`

## 10. Recurring Obligations

### Purpose

Maintain required outgoing events that must be protected in safe-to-spend logic.

### Access

- `admin`

### Users Can See

- list of recurring obligations
- cadence, amount, and next due date
- late or missing flags
- confidence or drift indicators

### Users Can Do

- create an obligation
- edit cadence, amount, or next due date
- deactivate or skip an obligation
- inspect its impact on forecast safety

### Next Navigation

- `Admin Overview`
- `Reconciliation Queue`
- `Forecast Details`

## 11. Reconciliation Queue

### Purpose

Review uncertain matches between actual transactions and expected income or
obligations.

### Access

- `admin`

### Users Can See

- matched items
- review items
- missing items
- ignored items
- confidence and reason codes

### Users Can Do

- accept a suggested match
- reject a match
- mark an expected event as skipped or ignored
- override a system judgment
- trigger recalculation of forecast state

### Key States

- no review items
- pending review items
- high-risk missing items

### Next Navigation

- `Admin Overview`
- `Forecast Details`
- `Alerts`

## 12. Budget Settings

### Purpose

Define discretionary constraints that can inform safe spending guidance.

### Access

- `admin`

### Users Can See

- configured budget targets
- current discretionary consumption against plan
- categories or groups under watch

### Users Can Do

- create or edit budget targets
- enable or disable budgeting as an input to guidance
- inspect whether category overspend is affecting decisions

### Next Navigation

- `Admin Overview`
- `Forecast Details`

## 13. Audit Trail

### Purpose

Show the important events that explain how the system reached its current state.

### Access

- `admin`

### Users Can See

- auth events
- config changes
- reconciliation overrides
- important forecast-generation events

### Users Can Do

- inspect recent changes
- diagnose why the forecast or trust state changed
- trace important actions back to a user or system event

### Next Navigation

- `Admin Overview`
- related admin pages based on the event being reviewed

## 14. User Access

### Purpose

Control which household users can authenticate and what role they have.

### Access

- `admin`

### Users Can See

- allowlisted users
- current role assignments
- access status

### Users Can Do

- add or remove an allowlisted user
- change a role
- disable access
- confirm that no public signup path exists

### Next Navigation

- `Admin Overview`

## Coverage Check

This page inventory should stay aligned with:

- `09_ux_flows.md` for navigation and flow shape
- `03_personas_and_scenarios.md` for user intent
- `05_functional_requirements.md` for page behavior expectations
