# Architecture Overview

This is a **living document** — it describes the architecture as it
exists *now*, and is rewritten (not appended to) as each phase in
`docs/roadmap.md` lands. For the history of *why* a given decision was
made, see `docs/adr/`. For terms used below, see `glossary.md`.

## Current state (end of Phase 0)

No application code exists yet. What exists is: a multi-module Maven
skeleton (one empty module per component below) and this documentation
set. Every statement below describes intended architecture to be built,
not yet-running behavior.

## System context

The system has one human-facing entry point (not yet built) that issues
requests expressing intent ("pay invoice #123", "check account balance").
Everything downstream of that entry point is the architecture described
here. There is no external network integration in this project — all
"enterprise systems" the agent touches are the synthetic
`payment-domain-service`.

## Components and boundaries

```
                     ┌─────────────────────┐
                     │   ai-gateway         │  (Phase 6)
                     │  (provider broker)   │
                     └──────────▲───────────┘
                                │ model calls only
                     ┌──────────┴───────────┐        ┌──────────────────────┐
   user/business  →  │   agent-runtime      │  ←───  │ enterprise-context    │ (Phase 7)
   intent             │   (Phase 2)          │  reads │ (read-only RAG)      │
                     └──────────┬───────────┘        └──────────────────────┘
                                │ tool calls only (no direct domain access)
                     ┌──────────▼───────────┐
                     │  tool-governance      │  (Phase 3)
                     │  (policy checkpoint)  │
                     └────┬─────────────┬────┘
                          │ allow        │ escalate
                          ▼              ▼
              ┌───────────────────┐  ┌─────────────────┐
              │ payment-domain-   │  │ human-approval    │ (Phase 4)
              │ service (Phase 1) │  │ (pause/resume)    │
              └───────────────────┘  └─────────────────┘

   observability (Phase 5) instruments agent-runtime, tool-governance,
   and human-approval — cross-cutting, not shown as a call path.

   evaluation (Phase 8) runs outside this diagram entirely, exercising
   agent-runtime from the outside as a test consumer.
```

## The load-bearing boundary

The single rule every later phase must preserve (ADR-0004):
**`agent-runtime` never calls `payment-domain-service` directly.**
Every path from "agent" to "money moves" passes through
`tool-governance`. This is what makes the rest of the phases
(approval, observability, gateway) meaningful — each of them is a control
attached to that one choke point, not scattered checks across the
codebase.

## Component reference

See each module's own `README.md` for its specific purpose, phase, and
"what it will NOT contain" boundary:

- `payment-domain-service` — synthetic system of record (Phase 1)
- `agent-runtime` — Spring AI agent orchestration (Phase 2)
- `tool-governance` — tool schema + policy checkpoint (Phase 3)
- `human-approval` — durable pause/resume for risky actions (Phase 4)
- `observability` — shared tracing/logging for agent behavior (Phase 5)
- `ai-gateway` — model provider broker/control plane (Phase 6)
- `enterprise-context` — read-only retrieval/RAG (Phase 7)
- `evaluation` — offline/online agent behavior scoring (Phase 8)

## Updating this document

When a phase lands: update the diagram/prose above to reflect what is
actually built (move it from "intended" to "current"), and cross-check
whether `docs/mvp-vs-enterprise.md` needs a new row.
