# Definition Of Done

## Feature Level

A feature is done when:

- the problem and user outcome are documented in the relevant feature dossier
- functional requirements and acceptance criteria are traced to the feature
- the UX flow impact is reflected where necessary
- data model and operational assumptions are explicit
- required test outlines identify what must be validated before release
- security and auth implications are addressed when applicable
- admin and non-technical household experiences are both considered

## Planning Artifact Level

A workflow artifact is done when:

- it references its source canonical docs or feature dossier
- it contains enough detail to be turned into engineering work without guesswork
- it clearly separates must-have scope from future hardening
- it uses the expected template or format for its target output

## Release Slice Level

A release slice is done when:

- the critical forecast and safe-to-spend behavior works end to end
- regressions that could damage trust are covered by tests or explicit test outlines
- unapproved users cannot authenticate
- the non-technical user can reach the key answer quickly
- the admin user can diagnose important mismatches or stale-state issues

## Product Level

BalanceIt3 is ready for household use when:

- both intended users can sign in successfully and no other users can
- the dashboard reliably surfaces `safe to spend`, recent transactions, and forecast highlights
- recurring obligations and income materially improve forecast quality
- explanation and warning states make wrong or stale outputs visible
- the technical household operator can maintain the system without excessive manual effort
