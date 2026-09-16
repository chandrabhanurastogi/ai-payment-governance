# Roadmap

Each phase is delivered as one or more commits/PRs on `main`, in order.
A phase is not started until the previous phase's exit criteria are met.
Each phase that introduces a new architectural decision adds its own
ADR(s) to `docs/adr/` and, if it introduces a simplification, a row to
`docs/mvp-vs-enterprise.md`.

## Phase 0 — Architecture and ADRs (this phase)

**Goal:** fix the shape of the system and the ground rules before any
application code exists.

**Exit criteria:**
- Component boundaries documented (`docs/architecture/overview.md`)
- Foundational ADRs accepted (0001–0005)
- Roadmap and MVP-vs-enterprise ledger exist
- Empty multi-module Maven skeleton exists, with no dependencies or
  source code in any module

## Phase 1 — Payment domain

**Goal:** a real, rule-bearing synthetic payment system of record.

**Exit criteria:** `payment-domain-service` has accounts, transactions,
and a ledger with real invariants (e.g. balances can't go negative,
transactions are atomic), exposed via a REST API, with tests. No AI
involved yet - this module should be understandable with zero AI context.

## Phase 2 — First AI agent

**Goal:** an agent that can converse and call tools, without any
governance yet (governance is deliberately deferred to Phase 3 so its
absence is felt first).

**Exit criteria:** `agent-runtime` uses Spring AI to hold a conversation
and invoke at least one tool that reaches `payment-domain-service`
*through* a placeholder in `tool-governance` (even if that placeholder is
a pass-through) - so the boundary from ADR-0004 exists in code from the
first line of agent logic, even before real policy is added.

## Phase 3 — Tool governance

**Goal:** the pass-through from Phase 2 becomes a real policy checkpoint.

**Exit criteria:** `tool-governance` declares tool schemas explicitly,
enforces at least one real policy (e.g. a payment amount limit), and
denies out-of-policy calls with a clear, logged reason.

## Phase 4 — Human approval

**Goal:** policy violations don't just get denied - some get escalated.

**Exit criteria:** `tool-governance` can route a call to
`human-approval` instead of an outright allow/deny; the agent's turn
suspends and later resumes correctly after an approval decision, even
across a restart.

## Phase 5 — Observability

**Goal:** every agent decision, tool call, and approval is traceable end
to end.

**Exit criteria:** `observability` provides shared instrumentation used by
`agent-runtime`, `tool-governance`, and `human-approval`; a single agent
turn can be reconstructed from logs/traces after the fact.

## Phase 6 — AI gateway

**Goal:** centralize and control access to the model provider.

**Exit criteria:** `agent-runtime` no longer holds a model provider
credential directly - all model calls go through `ai-gateway`, which logs
and can rate-limit/cap them.

## Phase 7 — Enterprise context / RAG

**Goal:** the agent can ground its answers in enterprise documents.

**Exit criteria:** `enterprise-context` retrieves relevant text for a
query and supplies it to the agent as context; retrieval is read-only and
cannot itself trigger a domain action.

## Phase 8 — AI evaluation

**Goal:** changes to prompts/models/tools are regression-tested.

**Exit criteria:** `evaluation` runs a golden scenario set against the
agent and reports pass/fail, runnable in CI, demonstrating at least one
caught regression during development.
