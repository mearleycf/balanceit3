# Data Ingestion

## Purpose

Data ingestion allows transactions to enter the system through CSV imports or connected sources while preserving lineage and avoiding double counting.

## Core Capabilities

- validate CSV uploads
- normalize imported transactions
- preserve source references
- deduplicate imported and synced activity
- route questionable records to review

## Key Requirements

- `FR-7` through `FR-10`
- `FR-30`

## Risks

- duplicate transactions distort forecast outputs
- broken lineage weakens trust and diagnosis
- poor validation creates hidden gaps in history
