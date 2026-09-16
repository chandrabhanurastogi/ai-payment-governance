# Roadmap

Each phase is delivered as one or more commits/PRs on `main`, in order.
A phase is not started until the previous phase's exit criteria are met.
Each phase that introduces a new architectural decision adds its own
ADR(s) to `docs/adr/` and, if it introduces a simplification, a row to
`docs/mvp-vs-enterprise.md`.

Components named below (`domain`, `agent`, `governance`, `approval`,
`observability`, `gateway`, `context`, `eval`) are **packages within one
Spring Boot application**, not separate modules or services — see
ADR-0007. A phase's job is to build the package, not to stand up a new
deployable.

The long-term vision for this project is
`prompts/full_project_specification.md`. Each phase below reconciles only
the slice of that vision relevant to it — see `CLAUDE.md` for how the
specification, this roadmap, and the ADRs relate.

## Phase 0 — Architecture and ADRs (this phase)

**Goal:** fix the shape of the system and the ground rules before any
application code exists.

**Exit criteria:**
- Component boundaries documented as logical/package boundaries
  (`docs/architecture/overview.md`)
- Foundational ADRs accepted (0001, 0003–0007; 0002 superseded by 0007)
- Roadmap and MVP-vs-enterprise ledger exist
- A lightweight threat model exists (`docs/security/threat-model.md`),
  mapping major threats to the phase expected to address them
- No build files or source code exist yet - Phase 0 is documentation only

## Phase 1 — Payment domain

**Goal:** a synthetic payment system of record broad enough to support
meaningful investigation scenarios later, without becoming separate
services or elaborate mini-systems.

**Exit criteria:** a single-module Spring Boot application exists with a
`domain` package covering payment, fraud/risk, authorization, settlement,
and incident concepts as intentionally small sub-areas of that one
package (e.g. `domain.payment`, `domain.fraud`, `domain.authorization`,
`domain.settlement`, `domain.incident`) - not separate services, and not
five fully-fledged systems. Each sub-area should have just enough
realistic behavior (a handful of states, one or two real invariants) to
give the Investigation Agent (Phase 2) something genuine to correlate
across - depth is intentionally shallow everywhere except the core
payment/transaction/ledger invariants (e.g. balances can't go negative,
transactions are atomic). Exposed via a REST API, with tests. No AI
involved yet - this package should be understandable with zero AI
context.

## Phase 2 — First AI agent

**Goal:** a first, concrete agent - the Payment Investigation Agent -
that can converse and call tools. `governance` exists only as a
*structural* boundary in this phase, not yet a substantive policy
checkpoint - its absence of real enforcement is deliberate, so that
absence is felt before Phase 3 fills it in.

**Exit criteria:** an `agent` package uses Spring AI to implement the
**Payment Investigation Agent**: it holds a conversation and invokes at
least one tool that reaches `domain` *through* `governance`, where
`governance` is present only as a structural pass-through/placeholder
(it exists in code and is the only path from `agent` to `domain`, but it
does not yet evaluate any real policy). This means the boundary from
ADR-0004 exists in code from the first line of agent logic, even though
no substantive enforcement exists yet - Phase 3's job is to turn this
placeholder into real enforcement, not to introduce the boundary itself.

**Design constraint:** the agent depends on Spring AI's model abstraction
(e.g. `ChatModel`), not a concrete provider SDK, so the model provider
can be swapped later without changing agent logic. The specific
implementation approach is recorded in a Phase 2 ADR when this is built -
not decided in Phase 0.

## Phase 3 — Tool governance

**Goal:** the structural pass-through from Phase 2 becomes a real policy
checkpoint, and the Identity gap (ADR-0006) gets its first owner.

**Exit criteria:** `governance` declares tool schemas explicitly,
enforces at least one real policy (e.g. a payment amount limit), and
denies out-of-policy calls with a clear, logged reason. This phase also
introduces **agent identity and basic RBAC**: every tool call is
attributable to a specific agent identity and a role, and policy can
allow/deny based on that role. An architecture test (e.g. ArchUnit)
enforcing "agent never calls domain directly" is added here or in Phase
2, whichever first has real code to check.

## Phase 4 — Human approval

**Goal:** policy violations don't just get denied - some get escalated.

**Exit criteria:** `governance` can route a call to `approval` instead of
an outright allow/deny; the agent's turn suspends and later resumes
correctly after an approval decision, even across a restart.

## Phase 5 — Observability

**Goal:** every agent decision, tool call, and approval is traceable end
to end.

**Exit criteria:** `observability` provides shared instrumentation used by
`agent`, `governance`, and `approval`; a single agent turn can be
reconstructed from logs/traces after the fact.

## Phase 6 — AI gateway

**Goal:** centralize and control access to the model provider.

**Exit criteria:** `agent` no longer holds a model provider credential
directly - all model calls go through `gateway`, which logs and can
rate-limit/cap them. This phase includes a **focused gateway comparison
ADR** evaluating 2-3 realistic options for this project (e.g. a custom
Spring Boot proxy, Spring AI's built-in abstraction, and one dedicated
gateway product such as LiteLLM) against the criteria that matter here
(auth, cost/rate control, logging, provider abstraction) - not an
exhaustive enterprise vendor evaluation across every cloud provider's AI
gateway offering.

## Phase 7 — Enterprise context / RAG

**Goal:** the agent can ground its answers in enterprise documents.

**Exit criteria:** `context` retrieves relevant text for a query and
supplies it to the agent as context; retrieval is read-only and cannot
itself trigger a domain action.

## Phase 8 — AI evaluation

**Goal:** changes to prompts/models/tools are regression-tested, with
representative (not exhaustive) coverage.

**Exit criteria:** `eval` runs a golden scenario set against the agent and
reports pass/fail, runnable in CI, demonstrating at least one caught
regression during development. The golden set includes representative
cases from each of these categories, kept small and proportional to a
learning project: normal investigation behavior, an ambiguous/
under-specified request, an out-of-policy tool attempt (governance should
deny it), and one adversarial input (e.g. a basic prompt-injection
attempt) showing governance holds regardless of what the LLM "decides."

## Beyond Phase 8 — extraction, if ever warranted

Not a scheduled phase. If a package outgrows being a package (per
ADR-0007's promotion rule - a genuine need for independent testing,
deployability, or ownership), splitting it into its own Maven module or
service is a deliberate, ADR-recorded decision made at that point, not a
default end state for this project.
