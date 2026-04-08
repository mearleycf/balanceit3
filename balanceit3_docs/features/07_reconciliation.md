# Reconciliation

## Purpose

Reconciliation compares expected financial events to actual transactions so the system can trust, downgrade, or revise forecast assumptions.

## Core Capabilities

- match transactions to expected income and obligations
- classify outcomes into matched, review, missing, or ignored
- support manual admin override
- feed confidence and warning state back into forecasting

## Key Requirements

- `FR-16` through `FR-19`
- `FR-21`
- `FR-31`

## Trust Impact

Reconciliation is the bridge between expected future state and observed reality. Without it, forecast confidence degrades quickly.
