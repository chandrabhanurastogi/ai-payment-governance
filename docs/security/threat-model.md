# Threat Model (Lightweight)

**Status:** Phase 0 documentation only. No security controls are
implemented yet - each threat below is mapped to the phase expected to
address it.

This is a lightweight, living threat model for a learning project, not a
formal enterprise threat-modeling exercise (no STRIDE workshop, no red
team). Its purpose is to make explicit, from the start, which risks exist
and which phase is expected to address them - consistent with
`docs/mvp-vs-enterprise.md`'s discipline of naming shortcuts rather than
hiding them.

## How to read this document

For each threat: what could go wrong, the primary mitigation this project
will build, the phase expected to address it, and the residual risk we
consciously accept until then (and often after, given this is a learning
project, not a production system).

## Threats

| # | Threat | Primary mitigation | Phase | Residual risk in this project |
|---|---|---|---|---|
| 1 | Agent bypasses governance and calls the payment domain directly | Trust boundary: `agent` never calls `domain` directly (ADR-0004) | 2 (boundary exists in code) / 3 (enforced by an architecture test) | No compiler/network enforcement until the ArchUnit test lands; a determined change could still bypass it |
| 2 | Agent invokes a tool it should not be allowed to use | Tool schema registry + policy enforcement (`governance`) | 3 | Policy is hand-written, not a mature policy-as-code engine; coverage is only as good as the rules we write |
| 3 | No way to attribute a tool call to a specific agent or role | Agent identity + basic RBAC | 3 | No real IdP/OIDC integration; identity is a simple attribute, not cryptographically verified |
| 4 | Sensitive payment data (e.g. card numbers) exposed in responses, logs, or LLM context | Data classification + data minimization | 3 (policy) / 7 (context) | No automated PII/secret scanning; correctness depends on manual review of what each tool returns |
| 5 | Prompt injection: malicious content (in user input or retrieved context) manipulates the agent into requesting an unauthorized action | Governance gates every tool call regardless of what the LLM "decides"; one representative adversarial test case | 3 (governance holds regardless) / 8 (regression test) | No dedicated injection-detection tooling; relies on governance being the actual authority, not on catching the injection itself |
| 6 | Malformed or unexpected tool-call arguments reach the domain | Input validation against declared tool schemas | 3 | Schemas are hand-rolled and reviewed, not exhaustively fuzz-tested |
| 7 | AI automatically executes a sensitive operation (refund, cancel) without human sign-off | Human-in-the-loop approval workflow | 4 | Single-approver model only; no multi-party maker-checker, no SLA escalation |
| 8 | Incomplete or missing audit trail for what an agent did and why | Structured observability/audit instrumentation | 5 | Retention/redaction policy is basic; no dedicated audit platform or long-term retention guarantees |
| 9 | Runaway agent loop or unexpected cost from uncontrolled model usage | AI gateway rate limiting / cost tracking | 6 | Static limits only; no real-time billing integration or anomaly detection |
| 10 | Agent retrieves and echoes back context/documents it should not have access to | Read-only, separate context boundary (`context` package) | 7 | No per-user access-control-aware retrieval in the MVP; one shared corpus |
| 11 | A change to a prompt, model, or tool silently regresses safe/correct behavior | Golden-scenario evaluation harness | 8 | Small, hand-curated scenario set - not statistically robust, won't catch every regression |
| 12 | Secrets or credentials committed to source or written to logs | "Never put secrets in source" principle (`CLAUDE.md`); externalized config | All phases | Enforced by discipline and review, not by an automated secret-scanning CI gate (not yet in scope) |

## Explicitly out of scope for this project

Consistent with `docs/mvp-vs-enterprise.md`: real network segmentation/
service mesh, a production IdP/OIDC deployment, a policy-as-code engine
(OPA/Rego), a formal DLP/data-classification program, and red-team-grade
adversarial testing. These are named here as the enterprise-grade version
of controls this project only demonstrates the *shape* of.

## Updating this document

Revisit this table when a phase lands: move the addressed threat's
residual-risk description to reflect what was actually built, and add
any new threat a phase's design surfaces that wasn't anticipated here.
