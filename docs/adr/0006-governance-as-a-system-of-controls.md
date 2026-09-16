# 0006. Governance is a system of controls, not a single layer

Date: 2026-09-16

## Status

Accepted

## Context

ADR-0004 fixed the agent-to-domain trust boundary and, in doing so,
described `tool-governance` as "the single most important boundary in
the whole system." That framing was useful for making the point that the
boundary must be decided before any code exists, but taken literally it
overclaims: it suggests one component is responsible for "governance,"
when the roadmap already treats several other concerns as independent
control dimensions with their own phases (human approval, observability,
AI gateway, enterprise context, evaluation). It also silently said
nothing about two concerns - **identity** and **data classification** -
that `CLAUDE.md`'s own governance principles already name, and that don't
yet have an owner.

Treating governance as one interception point, rather than a set of
controls, risks two failure modes later: (a) overloading the governance
package with responsibilities that belong elsewhere (e.g. trying to make
it also own audit or approval), and (b) leaving identity and data
classification unaddressed because no phase's exit criteria ever mentions
them.

## Decision

We explicitly model governance as a system of (at least) the following
control dimensions, each mapped to the logical component that owns it
today (see ADR-0007 - these are packages within one application, not
separate modules or services), and we stop describing any single one of
them as "the most important":

| Control dimension | Owned by (logical component) | Question it answers |
|---|---|---|
| Authorization & Policy | governance | Is this specific tool call, right now, allowed at all? |
| Human Approval | approval | If policy alone can't decide, what does a human say? |
| Audit & Observability | observability | After the fact, what happened and why? |
| Model/Provider Controls | gateway | Is this call to the model itself within cost/rate/model-allowlist limits? |
| Context/Knowledge Controls | context | What is the agent allowed to *read*, and is that kept separate from what it can *act on*? |
| Behavior-over-time | eval | Has agent behavior regressed across a prompt/model/tool change? |
| **Identity** | *not yet owned* | Who is the human user, and who/what is the agent acting as? |
| **Data Classification** | *not yet owned* | What sensitivity level does this piece of data carry, and does that change what policy/audit/context rules apply to it? |

The governance component's actual job is narrower than "governance" as a
whole: it is authorization + policy enforcement specifically at the
agent-to-domain tool boundary. ADR-0004's decision (the agent package
never calls the domain package directly) is unchanged by this ADR - only
the description of how central that one component is has been corrected.

Identity and data classification are named here as **explicit gaps**,
not assigned to an existing component by default. Whether they become
their own package, get folded into an existing one, or are handled
outside the codebase entirely (e.g. identity via a real IdP even in the
learning setup) is deferred to a future ADR, written when a phase first
needs one of them - likely identity alongside Phase 3/4 (a tool call and
an approval both need to know *who* is asking) and data classification
alongside Phase 3 or Phase 7 (policy and retrieval both need to know
*what* they're handling).

## Consequences

**For this learning MVP:** `docs/architecture/overview.md` describes
governance as controls surrounding the agent across multiple phases, not
a single box the agent passes through once. Identity and data
classification are tracked as open gaps in that document rather than
quietly assumed to be someone else's problem.

**For a real enterprise:** these dimensions are almost never one team's
responsibility - identity typically comes from an enterprise IdP/IAM
platform, data classification from a DLP/data-governance program, audit
from a central logging/SIEM platform, and so on. Naming them separately
here, even before any of them has code, is what will let later phases map
each one onto a plausible real system instead of inventing a bespoke one.

## Notes

Refines the framing introduced in ADR-0004; does not change ADR-0004's
decision. See the amendment note at the end of ADR-0004.
