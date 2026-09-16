# Observability

**Phase:** 5 — Observability
**Status:** empty skeleton (no source code yet)

## Purpose

Shared instrumentation used by the other modules to record what an agent
did: prompts, tool-call arguments and results, policy decisions, and
approval outcomes - not just latency and error rates.

## Why this is a separate module (and why it's a library, not a service)

Unlike the other components, this one is cross-cutting rather than a
service boundary: it is consumed by `agent-runtime`, `tool-governance`,
and `human-approval` alike. It is kept as its own module so that
"AI-specific" observability concerns (capturing reasoning/tool-call
content, not just metrics) are defined once and reused, instead of each
module inventing its own logging shape.

## What it will NOT contain

Business or agent logic - only instrumentation utilities and the
conventions other modules depend on.
