# Architecture Overview

This is a **living document** — it describes the architecture as it
exists *now*, and is rewritten (not appended to) as each phase in
`docs/roadmap.md` lands. For the history of *why* a given decision was
made, see `docs/adr/`. For terms used below, see `glossary.md`.

## Current state (end of Phase 0)

No application code exists yet, and no build files exist yet either —
Phase 0 is documentation only (see ADR-0007). Everything below describes
architecture *intended* for Phase 1 onward, not yet-running behavior.

A lightweight threat model (`docs/security/threat-model.md`) exists
alongside this document, mapping major risks to the phase expected to
address them.

The components below are **logical boundaries**, not separate
deployables. Starting in Phase 1 they are built as **packages inside one
Spring Boot application**, sharing a single `pom.xml` and a single
runtime process. A component is only promoted to its own Maven module —
and, later, only a module is considered for promotion to its own
service — when a concrete, named need arises (see ADR-0007's promotion
rule). Until then, treat every box below as "a package," not "a service."

## System context

The system has one human-facing entry point (not yet built) that issues
requests expressing intent ("pay invoice #123", "check account balance").
Everything downstream of that entry point is the architecture described
here. There is no external network integration in this project — all
"enterprise systems" the agent touches are the synthetic payment domain
built in Phase 1.

## Logical components and boundaries

```
                     ┌─────────────────────┐
                     │   gateway            │  (Phase 6)
                     │  (provider broker)   │
                     └──────────▲───────────┘
                                │ model calls only
                     ┌──────────┴───────────┐        ┌──────────────────────┐
   user/business  →  │   agent               │  ←───  │ context               │ (Phase 7)
   intent             │   (Phase 2)          │  reads │ (read-only RAG)      │
                     └──────────┬───────────┘        └──────────────────────┘
                                │ tool calls only (no direct domain access)
                     ┌──────────▼───────────┐
                     │  governance           │  (Phase 3)
                     │  (policy checkpoint)  │
                     └────┬─────────────┬────┘
                          │ allow        │ escalate
                          ▼              ▼
              ┌───────────────────┐  ┌─────────────────┐
              │ domain            │  │ approval          │ (Phase 4)
              │ (Phase 1)         │  │ (pause/resume)    │
              └───────────────────┘  └─────────────────┘

   observability (Phase 5) instruments agent, governance, and approval —
   cross-cutting, not shown as a call path.

   eval (Phase 8) runs outside this diagram entirely, exercising the
   agent package from the outside as a test consumer.

   All boxes above are packages within one Spring Boot application
   (ADR-0007), not separate deployables.
```

## The trust boundary, and why it isn't the whole story

The one rule every later phase must preserve (ADR-0004): **the `agent`
package never calls the `domain` package directly.** Every path from
"agent" to "money moves" passes through `governance`. For the MVP this is
enforced by code review, with an architecture test (e.g. ArchUnit) to be
added once there's real code to enforce it (see ADR-0007) — there is no
process/network boundary backing it, since everything runs in one JVM.

That boundary is necessary, but it is not the same thing as
"governance." Per ADR-0006, governance is a system of controls, each
answering a different question, mapped to the logical component that
owns it:

| Control | Owned by (package) | Question it answers |
|---|---|---|
| Authorization & Policy | governance | Is this tool call allowed at all? |
| Human Approval | approval | If policy can't decide alone, what does a human say? |
| Audit & Observability | observability | After the fact, what happened and why? |
| Model/Provider Controls | gateway | Is this call to the model itself within limits? |
| Context/Knowledge Controls | context | What can the agent read, kept separate from what it can act on? |
| Behavior-over-time | eval | Has behavior regressed across a change? |
| **Identity** *(gap)* | not yet owned | Who is the user, and who/what is the agent acting as? |
| **Data Classification** *(gap)* | not yet owned | What sensitivity does this data carry? |

`gateway` governs the *model* call path; `governance` governs the
*domain* call path — these are two separate boundaries, not one. Identity
and data classification are tracked here as open gaps rather than
silently assumed to live inside `governance`; see ADR-0006 for when each
is expected to get an owner.

## Component reference

| Package (illustrative name) | Phase | Purpose | Will NOT contain |
|---|---|---|---|
| `domain` | 1 | Synthetic payment system of record: payment, fraud/risk, authorization, settlement, and incident concepts as small sub-areas, with real invariants concentrated in payment/transaction/ledger | Any AI/agent/LLM logic; five elaborate mini-systems - depth stays shallow outside the core invariants |
| `agent` | 2 | Spring AI conversation loop; turns intent into tool calls | Direct calls into `domain` |
| `governance` | 3 | Tool schema registry + policy enforcement at the agent→domain boundary | Agent/LLM logic; payment business logic |
| `approval` | 4 | Durable pause/resume for actions `governance` escalates to a human | The policy decision of *whether* to escalate |
| `observability` | 5 | Shared tracing/logging conventions for prompts, tool calls, decisions | Business or agent logic |
| `gateway` | 6 | Broker in front of the model provider(s): credentials, rate limits, cost, logging | Agent orchestration logic |
| `context` | 7 | Read-only retrieval of policy/product/case documents | Any write path back into `domain` |
| `eval` | 8 | Offline/online scoring of agent behavior against golden scenarios | Anything the running agent depends on at request time |

Exact package names will be finalized when each phase actually creates
them — the table above is the current intent, not a commitment.

## Relationship to the full project specification

`prompts/full_project_specification.md` describes the long-term vision
for this system (see `CLAUDE.md` for how it relates to the ADRs and this
roadmap). Two places where this document deliberately narrows or diverges
from that specification's illustrative detail:

- **"AI Gateway" scope.** The specification's diagram (§4) places an "AI
  Gateway / AI Control Plane" at the very front of the system, combining
  entry routing with governance dispatch. This document instead keeps
  `gateway` narrowly scoped to the model-provider call path only
  (ADR-0006) - a single-responsibility component is more consistent with
  ADR-0006's decomposition than one component absorbing "all of AI
  governance."
- **Repository structure.** The specification's example repository
  layout (§20) proposes separate services per domain capability and per
  platform component. ADR-0007's promotion rule already answers the
  question that section poses ("which components should be independently
  deployable"): none of them yet has a concrete reason to be, so all
  remain packages in one application.

## Updating this document

When a phase lands: update the diagram/prose above to reflect what is
actually built (move it from "intended" to "current"), and cross-check
whether `docs/mvp-vs-enterprise.md` needs a new row.
