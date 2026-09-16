# 0004. Separate the agent runtime from the payment domain via a governed tool layer

Date: 2026-09-16

## Status

Accepted

## Context

This is the single most consequential architectural decision in the whole
project, and it needs to be fixed *before* any code exists so it can't be
quietly eroded later by a convenient shortcut ("just call the repository
directly, it's faster"). The core question: how does an AI agent get
access to do things in the payment domain?

Left unconstrained, an agent framework will happily be wired directly to
a database or an internal service client, because that's the path of
least resistance during a demo. That is also precisely the pattern that
makes "AI governance" impossible after the fact - if the agent can act on
the domain through more than one path, there is no single place to apply
policy, logging, or approval.

## Decision

The `agent-runtime` module **never** calls `payment-domain-service`
directly - no shared database, no direct REST/client call, no shared
JPA entities. Every action the agent can take on the payment domain is
exposed as an explicit **tool** with a declared schema, and every tool
call is routed through `tool-governance`, which is the only component
allowed to call into `payment-domain-service`.

`tool-governance` is responsible for: knowing the full set of tools that
exist, validating call arguments against policy (limits, allowed
currencies/operations), and deciding whether a call proceeds, is denied,
or is routed to `human-approval` (Phase 4).

## Consequences

**For this learning MVP:** enforced by module boundaries and code review
only - there is no network segmentation, so a determined contributor
*could* bypass it by adding a direct dependency. We accept this because
the goal at this stage is to make the boundary visible and intentional in
the codebase, not to make it physically unbypassable.

**For a real enterprise:** this boundary would be enforced at the
infrastructure level too - separate deployables, network policy or
service mesh rules that make `payment-domain-service` unreachable from
`agent-runtime`'s network identity, and mTLS/service-to-service auth so
only `tool-governance` holds a valid client credential for the domain
service. The tool layer would likely be its own deployed service (not
just a library dependency) precisely so it can't be bypassed even by a
misconfigured agent process.

## Notes

This ADR is a prerequisite for Phase 3 (tool-governance) and Phase 4
(human-approval) - both phases exist because this boundary exists. If a
future phase finds a reason to relax this rule, it must be done via a new
ADR that explicitly supersedes this one, not silently.

**Amendment (ADR-0006):** this ADR's decision (the trust boundary itself)
stands unchanged, but its Context section overstated `tool-governance`'s
centrality relative to the *other* governance controls (identity,
approval, audit, model/provider controls, context controls, evaluation).
See ADR-0006 for the corrected framing: governance as a system of
controls, not a single choke point.

**Amendment (ADR-0007):** where this ADR says "module" (e.g.
`agent-runtime` module, `tool-governance` module), read that as the
*logical component*, not a Maven module. ADR-0007 supersedes ADR-0002's
multi-module build and implements these components as packages within a
single application instead. The trust-boundary rule itself - the agent
package never calls the domain package directly, everything routes
through the governance package - is unchanged; only the build/deployment
unit changed.
