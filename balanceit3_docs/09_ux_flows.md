# UX Flows

## Design Principle

The UX should put the answer first. The non-technical user should see `safe to spend` immediately, while the admin user can drill into forecast logic, reconciliation state, and system health.

## Primary Household Journey

```mermaid
flowchart LR
    A["Sign in with allowlisted Google account"] --> B["Open dashboard"]
    B --> C["See safe to spend amount"]
    C --> D["Review supporting forecast summary"]
    D --> E["Inspect recent transactions"]
    D --> F["Inspect upcoming income and obligations"]
    D --> G["Notice warning or confidence badge if present"]
    G --> H["Read plain-language explanation"]
```

## Admin Reconciliation Journey

```mermaid
flowchart LR
    A["Admin opens forecast diagnostics"] --> B["Review low-confidence or missing-event warning"]
    B --> C["Open reconciliation queue"]
    C --> D["Inspect matched, review, missing, or ignored items"]
    D --> E["Accept, reject, or override item"]
    E --> F["Recalculate forecast"]
    F --> G["Verify updated safe to spend explanation"]
```

## New Data Source Journey

```mermaid
flowchart LR
    A["Admin opens account or import setup"] --> B["Connect account or upload CSV"]
    B --> C["Validate source data"]
    C --> D["Normalize and deduplicate transactions"]
    D --> E["Review import results"]
    E --> F["Include source in forecast"]
    F --> G["Updated dashboard reflects broader coverage"]
```

## Site Map

![Balanceit3_Sitemap](assets/Balanceit3_Sitemap.png)


## UX Notes

- `Dashboard` is the default landing page for both users.
- `Admin` sections should be hidden from the non-technical household user by default.
- Warning copy should favor plain language first and technical detail second.
- Confidence indicators should explain why the number is trustworthy or conservative.
