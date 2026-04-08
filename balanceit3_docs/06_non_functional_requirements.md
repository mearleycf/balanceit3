# Non-Functional Requirements

## Hosting And Operations

- `NFR-1`: The system must be deployable to lightweight hosting platforms such as Vercel or Hostinger without requiring a heavy operations footprint.
- `NFR-2`: The system should prefer managed services and low-maintenance infrastructure where that does not weaken forecast integrity or household privacy.
- `NFR-3`: Routine operation must be manageable by a single technical household operator.

## Security And Authentication

- `NFR-4`: Authentication must be limited to allowlisted Google OAuth identities for the two intended users.
- `NFR-5`: Any registration or sign-in attempt by an unapproved identity must be rejected by default.
- `NFR-6`: Protected data and admin functions must not be exposed to unauthenticated or non-allowlisted users.
- `NFR-7`: The system must use secure defaults for session handling, secrets management, and transport security suitable for an internet-accessible private app.
- `NFR-8`: Security requirements should focus on household-risk reduction rather than enterprise or public-sector compliance theater.

## Performance And Reliability

- `NFR-9`: The dashboard should load the current `safe to spend` result and supporting summary quickly enough for casual daily use on home internet and mobile devices.
- `NFR-10`: Forecast recalculation should complete fast enough that admin changes feel interactive rather than batch-oriented.
- `NFR-11`: The system must continue to present the last known forecast state when external sync dependencies are temporarily unavailable, while clearly marking stale data.

## Explainability And Trust

- `NFR-12`: Forecast and safe-to-spend outputs must be explainable in plain language for the non-technical user and in deeper diagnostic terms for the admin user.
- `NFR-13`: Low-confidence or stale conditions must reduce trust signaling explicitly rather than silently degrading results.
- `NFR-14`: Automated decisions that materially affect the forecast must leave an inspectable trail.

## Accessibility And Usability

- `NFR-15`: The interface should be readable and navigable on modern mobile and desktop devices without advanced training.
- `NFR-16`: Accessibility support should be practical and modern, but the product is not targeting formal certification or broad public-sector conformance programs.
- `NFR-17`: The non-technical user experience should minimize jargon, deep settings, and unnecessary configuration exposure.

## Auditability And Compliance Scope

- `NFR-18`: Audit coverage should focus on meaningful household and admin events such as login, configuration changes, forecast recalculation, and reconciliation overrides.
- `NFR-19`: The system does not need enterprise governance workflows or external regulatory audit preparation.
- `NFR-20`: Data lineage for transactions and forecast inputs must still be preserved because it directly affects trust in the `safe to spend` amount.
