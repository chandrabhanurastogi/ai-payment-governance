# 0007. Start as a single application; defer the multi-module split

Date: 2026-09-16

## Status

Accepted

Supersedes ADR-0002.

## Context

ADR-0002 committed to a multi-module Maven build with one top-level
module per architectural component, and Phase 0 scaffolded all eight
(`payment-domain-service`, `agent-runtime`, `tool-governance`,
`human-approval`, `observability`, `ai-gateway`, `enterprise-context`,
`evaluation`) as empty modules before any application code existed.

On review, this was premature for three reasons:

1. **It confused conceptual boundaries with build boundaries.** A Maven
   module is a real technical commitment - a separate compilation unit
   with its own dependency graph. Eight components existing only as a
   `pom.xml` and a README were not actually separate anything; they were
   documentation wearing build-tool clothing.
2. **It front-loaded a decision no phase yet needed.** Nothing in Phases
   1-4 requires independent deployability, independent scaling, or
   independent ownership - the actual reasons to split a module out. The
   split was speculative structure for hypothetical future requirements.
3. **It works against this project's own stated principles**
   (`CLAUDE.md`): *"Prefer the simplest architecture that demonstrates
   the capability"* and *"Do not create microservices merely to make the
   architecture look impressive."* Eight empty modules before a working
   payment domain existed was exactly that.

## Decision

We will build a **single Maven module / single Spring Boot application**
starting in Phase 1. The eight components identified in
`docs/architecture/overview.md` remain valid **logical** boundaries, and
the rules attached to them (most importantly ADR-0004's trust boundary)
still apply - but they are implemented as **Java packages within one
deployable**, not as separate Maven modules:

```
com.learning.paymentgovernance
├── domain          (Phase 1 - payment domain: accounts, transactions, ledger)
├── agent           (Phase 2 - agent runtime)
├── governance      (Phase 3 - tool authorization & policy)
├── approval        (Phase 4 - human approval workflow)
├── observability   (Phase 5 - shared tracing/logging)
├── gateway         (Phase 6 - model provider broker)
├── context         (Phase 7 - enterprise context / RAG)
└── eval            (Phase 8 - evaluation harness)
```

(Exact package names may be refined when each phase actually creates
them; the structure above is illustrative, not binding.)

**Promotion rule:** a package is only promoted to its own Maven module -
and, later, only a module is considered for promotion to its own
deployable/service - when a concrete, named need arises. Examples of a
concrete need: wanting to unit-test a package in isolation without
pulling in the rest of Spring Boot; wanting a compiler-enforced boundary
instead of a convention-enforced one; wanting independent deployability
because two components have genuinely different scaling or ownership
profiles. "It would look more like a real enterprise system" is
explicitly **not** a sufficient reason on its own.

Package boundaries (in particular: `agent` must never call `domain`
directly) are enforced by code review for now, with an architecture test
(e.g. ArchUnit) to be added in the phase that first needs it - likely
Phase 2 or 3 - rather than left to reviewer vigilance indefinitely.

## Consequences

**For this learning MVP:** one buildable, runnable application from the
first Phase 1 commit onward. Far less build ceremony (one `pom.xml`, one
`src/main/java` tree) while the boundaries are still being learned. The
boundaries remain visible and real - as packages and, later, as an
enforced architecture test - just not as premature build artifacts.

**For a real enterprise:** some of these components would still end up
as separate services eventually - `payment-domain-service`, `ai-gateway`,
and `evaluation` are the most likely candidates, since they plausibly
have different scaling, security, and ownership profiles from the rest.
The difference from ADR-0002's approach is sequencing: a real
architecture earns its service boundaries from concrete operational
requirements, not from an org chart drawn on day one.

## Notes

Alternative considered: keep multi-module but reduce to two or three
modules (e.g. `app` and `eval`). Rejected - even a reduced module count
still front-loads a build-topology decision before Phase 1 exists; the
simplest correct move is to defer the decision entirely until a package
outgrows being a package.
