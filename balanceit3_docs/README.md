# BalanceIt3 Documentation Workspace

This directory is a self-contained product requirements workspace for BalanceIt3.
It is designed for two audiences:

- humans making product and engineering decisions
- AI agents generating planning, delivery, and testing artifacts from those decisions

BalanceIt3 is the successor to BalanceIt2. This workspace uses BalanceIt2 as source material, but it defines the forward-looking product rather than mirroring the current implementation.

## What This Workspace Is

- a canonical product-spec package for the next version of the app
- a portable workspace that can be moved into its own repo or branch later
- a source for epics, stories, acceptance criteria, implementation plans, issue drafts, and testing outlines

## Primary Product Goal

The most important job of BalanceIt3 is to calculate a trustworthy `safe to spend` amount from near-term cash flow. Other features exist to improve the confidence, usability, and explainability of that number.

## Canonical Reading Order

1. [01_product_brief.md](./01_product_brief.md)
2. [02_domain_model.md](./02_domain_model.md)
3. [03_personas_and_scenarios.md](./03_personas_and_scenarios.md)
4. [04_capability_map.md](./04_capability_map.md)
5. [05_functional_requirements.md](./05_functional_requirements.md)
6. [06_non_functional_requirements.md](./06_non_functional_requirements.md)
7. [07_technical_guardrails.md](./07_technical_guardrails.md)
8. [08_definition_of_done.md](./08_definition_of_done.md)
9. [09_ux_flows.md](./09_ux_flows.md)
10. [10_page_inventory_and_actions.md](./10_page_inventory_and_actions.md)

Then read:

- [features/](./features/) for feature-level dossiers
- [history/](./history/) for provenance and divergence from BalanceIt2
- [workflows/](./workflows/) for downstream artifact generation

## What To Load By Task

| Task | Load These | Usually Skip |
| --- | --- | --- |
| Understand the product | `01_product_brief.md`, `03_personas_and_scenarios.md`, `04_capability_map.md` | workflow templates |
| Model data or APIs | `02_domain_model.md`, `05_functional_requirements.md`, relevant `features/*.md` | sprint packets |
| Plan implementation | `04_capability_map.md`, `05_functional_requirements.md`, `07_technical_guardrails.md`, relevant `features/*.md` | unrelated feature dossiers |
| Generate GitHub issues | relevant `features/*.md`, `05_functional_requirements.md`, `workflows/05_github_issue_drafts/` | full history unless provenance matters |
| Generate Linear issues | relevant `features/*.md`, `05_functional_requirements.md`, `workflows/06_linear_issue_drafts/` | UX flows unless story needs navigation context |
| Create a sprint packet | issue drafts, acceptance criteria, implementation plan, `workflows/07_sprint_packets/` | full domain model unless needed |
| Plan testing | relevant `features/*.md`, `05_functional_requirements.md`, `06_non_functional_requirements.md`, `workflows/08_testing/` | unrelated feature dossiers |
| Design UX or navigation | `03_personas_and_scenarios.md`, `04_capability_map.md`, `09_ux_flows.md`, `10_page_inventory_and_actions.md` | tracker-specific drafts |

## What Not To Load

- Do not load every file by default.
- Do not treat `history/` as canonical requirements.
- Do not use workflow drafts as the source of truth when a numbered canonical doc says otherwise.
- Do not let lower-priority features overtake `cash-flow forecasting` or `safe-to-spend calculation`.

## Folder Structure

```text
balanceit3_docs/
├── README.md
├── 01_product_brief.md
├── 02_domain_model.md
├── 03_personas_and_scenarios.md
├── 04_capability_map.md
├── 05_functional_requirements.md
├── 06_non_functional_requirements.md
├── 07_technical_guardrails.md
├── 08_definition_of_done.md
├── 09_ux_flows.md
├── 10_page_inventory_and_actions.md
├── features/
│   ├── 01_cash_flow_forecasting.md
│   ├── 02_safe_to_spend_calculation.md
│   └── ...
├── history/
│   ├── balanceit2_source_map.md
│   ├── decision_log.md
│   └── open_questions.md
└── workflows/
    ├── 01_epics/
    ├── 02_user_stories/
    ├── 03_acceptance_criteria/
    ├── 04_implementation_plans/
    ├── 05_github_issue_drafts/
    ├── 06_linear_issue_drafts/
    ├── 07_sprint_packets/
    └── 08_testing/
```

## File Placement Rules

- Top-level numbered files hold canonical cross-product decisions.
- `features/` holds one dossier per feature area. Do not combine unrelated features in one file.
- `history/` records source mapping, rationale, and unresolved questions.
- `workflows/` holds templates and dry-run examples that can be turned into delivery artifacts.

## Naming Conventions

- Canonical docs: `NN_name.md`
- Feature dossiers: `NN_feature_name.md`
- Templates: `template_<artifact>.md`
- Dry-run examples: `<feature>-<artifact>.md`
- Use lowercase snake case for filenames after the numeric prefix.

## Recommended Skills And Workflows

Use these as examples when an AI agent operates on this workspace:

- `brainstorming`: refine product or feature ideas before making new specs
- `writing-plans`: convert approved feature scope into detailed implementation plans
- `subagent-driven-development`: execute a plan in independent slices
- `verification-before-completion`: verify drafts and generated outputs before claiming they are ready
- `requesting-code-review`: review implementation artifacts generated from this workspace
- `linear`: convert issue-ready outputs into Linear tickets
- `conventional-commits-writer`: draft commit messages when this workspace is versioned

## Expected Outputs From This Workspace

- product epics
- user stories
- acceptance criteria packs
- implementation plans
- GitHub issue drafts
- Linear issue drafts
- sprint packets
- test strategy drafts
- scaffolded testing outlines that identify required suites, coverage targets, and validation intent
- edge-case inventories
- acceptance scenario drafts

## Traceability Rule

Use this flow and do not reverse it:

`canonical docs -> feature dossiers -> workflow artifacts -> issue drafts -> test outlines`

## Source Material

BalanceIt2 inputs are cataloged in [history/balanceit2_source_map.md](./history/balanceit2_source_map.md). If a new BalanceIt3 decision differs from BalanceIt2, record the divergence in [history/decision_log.md](./history/decision_log.md).
