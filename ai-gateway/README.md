# AI Gateway

**Phase:** 6 — AI gateway
**Status:** empty skeleton (no source code yet)

## Purpose

Sits between `agent-runtime` and the actual model provider(s). Centralizes
provider credentials, rate limiting, cost tracking, model routing, and
prompt/response logging.

## Why this is a separate module

No component should hold a raw provider API key or call a model provider
directly - that is what lets an enterprise apply one set of controls (cost
caps, allowed models, redaction) regardless of which team built the agent.
Introducing this late (Phase 6) is deliberate: it demonstrates retrofitting
a control plane onto an agent that was built first talking to a model
directly, which is a common real-world sequence.

## What it will NOT contain

Agent reasoning/orchestration logic - only provider abstraction and
governance controls around model calls.
