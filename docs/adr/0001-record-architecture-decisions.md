# 0001. Record architecture decisions

Date: 2026-09-16

## Status

Accepted

## Context

This project will make a series of architectural decisions over many
phases (repo layout, stack, trust boundaries, policy engine, workflow
engine, observability stack, gateway, RAG store, eval framework). Without
a record, the *reasoning* behind each decision is lost as soon as the
conversation that produced it scrolls away - only the resulting code
survives, and code doesn't explain why an alternative was rejected.

## Decision

We will record every significant architectural decision as a
Markdown Architecture Decision Record (ADR) in `docs/adr/`, numbered
sequentially (`0001`, `0002`, ...), using the template in
`docs/adr/template.md` (based on Michael Nygard's lightweight ADR format).

Rules:
- ADRs are **append-only**. Once `Accepted`, a record is never edited to
  reflect a later change of mind - a new ADR is written and the old one is
  marked `Superseded by ADR-XXXX`.
- One ADR = one decision. Don't bundle unrelated decisions.
- An ADR records *why*, not just *what* - the Context and Consequences
  sections are as important as the Decision itself.

## Consequences

**For this learning MVP:** a plain-text, git-tracked decision log that
lives next to the code, reviewable in the same PRs that implement the
decision. No tooling dependency (no `adr-tools` CLI needed).

**For a real enterprise:** the same lightweight format usually still
applies, but larger organizations often add: a required "ADR link" field
in PR templates, a lightweight approval step for ADRs above a certain risk
tier (e.g. anything touching data residency or customer funds movement),
and sometimes a searchable ADR index tool. We are not doing any of that
here - discipline is manual, not enforced.

## Notes

Alternative considered: keeping decisions only in commit messages / PR
descriptions. Rejected because those are hard to browse as a coherent set
once the number of decisions grows, and they get buried under
implementation-detail commits.
