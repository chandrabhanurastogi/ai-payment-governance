# Payment Domain Service

**Phase:** 1 — Payment domain
**Status:** empty skeleton (no source code yet)

## Purpose

The synthetic "system of record" that AI agents will operate against:
accounts, transactions, and a ledger, exposed as a business-rule-bearing
API rather than a raw CRUD store.

## Why this is a separate module

This module represents the enterprise system(s) that already exist before
any AI is introduced. In a real bank this would be many systems (core
banking, ledger, fraud, sanctions screening); here it is deliberately
collapsed into one Spring Boot service so the *behavior* (business rules,
invariants, failure modes) is real even though the surrounding landscape
is simplified.

It is kept in its own module - not merged into `agent-runtime` - because
the architecture depends on the agent never having direct access to it.
See ADR-0004 and `docs/architecture/overview.md`.

## What it will NOT contain

Any AI/agent/LLM logic. This module should be understandable and testable
by someone who has never heard of Spring AI.
