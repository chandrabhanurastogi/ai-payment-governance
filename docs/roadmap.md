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

## Phase 0 — Architecture and ADRs (this phase)

**Goal:** fix the shape of the system and the ground rules before any
application code exists.

**Exit criteria:**
- Component boundaries documented as logical/package boundaries
  (`docs/architecture/overview.md`)
- Foundational ADRs accepted (0001, 0003–0007; 0002 superseded by 0007)
- Roadmap and MVP-vs-enterprise ledger exist
- No build files or source code exist yet - Phase 0 is documentation only

## Phase 1 — Payment domain

**Goal:** a real, rule-bearing synthetic payment system of record.

**Exit criteria:** a single-module Spring Boot application exists with a
`domain` package containing accounts, transactions, and a ledger with
real invariants (e.g. balances can't go negative, transactions are
atomic), exposed via a REST API, with tests. No AI involved yet - this
package should be understandable with zero AI context.

## Phase 2 — First AI agent

**Goal:** an agent that can converse and call tools, without any
governance yet (governance is deliberately deferred to Phase 3 so its
absence is felt first).

**Exit criteria:** an `agent` package uses Spring AI to hold a
conversation and invoke at least one tool that reaches `domain`
*through* a placeholder in `governance` (even if that placeholder is a
pass-through) - so the boundary from ADR-0004 exists in code from the
first line of agent logic, even before real policy is added.

## Phase 3 — Tool governance

**Goal:** the pass-through from Phase 2 becomes a real policy checkpoint.

**Exit criteria:** `governance` declares tool schemas explicitly,
enforces at least one real policy (e.g. a payment amount limit), and
denies out-of-policy calls with a clear, logged reason. An architecture
test (e.g. ArchUnit) enforcing "agent never calls domain directly" is
added here or in Phase 2, whichever first has real code to check.

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
rate-limit/cap them.

## Phase 7 — Enterprise context / RAG

**Goal:** the agent can ground its answers in enterprise documents.

**Exit criteria:** `context` retrieves relevant text for a query and
supplies it to the agent as context; retrieval is read-only and cannot
itself trigger a domain action.

## Phase 8 — AI evaluation

**Goal:** changes to prompts/models/tools are regression-tested.

**Exit criteria:** `eval` runs a golden scenario set against the agent and
reports pass/fail, runnable in CI, demonstrating at least one caught
regression during development.

## Beyond Phase 8 — extraction, if ever warranted

Not a scheduled phase. If a package outgrows being a package (per
ADR-0007's promotion rule - a genuine need for independent testing,
deployability, or ownership), splitting it into its own Maven module or
service is a deliberate, ADR-recorded decision made at that point, not a
default end state for this project.
