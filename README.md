# AI Payment Governance

Learning project exploring enterprise AI governance,
AI agents, context management, tooling and guardrails
using a synthetic payment domain.

The project is being developed incrementally with
Java, Spring Boot and Spring AI, one architectural phase at a time.

## Start here

- [`docs/roadmap.md`](docs/roadmap.md) — the phase plan and exit criteria
- [`docs/architecture/overview.md`](docs/architecture/overview.md) — current-state architecture and component boundaries
- [`docs/architecture/glossary.md`](docs/architecture/glossary.md) — shared vocabulary
- [`docs/adr/`](docs/adr/) — architecture decision records
- [`docs/mvp-vs-enterprise.md`](docs/mvp-vs-enterprise.md) — what's simplified for learning vs. what a real enterprise needs

## Repository layout

Phase 0 is documentation only — no build files or source code exist yet.
Starting in Phase 1, this becomes a single Maven module / single Spring
Boot application (see ADR-0007, superseding an earlier multi-module plan
in ADR-0002). Architectural components are Java packages within that one
application, not separate modules or services, unless a concrete need
later justifies splitting one out:

- `domain` (Phase 1)
- `agent` (Phase 2)
- `governance` (Phase 3)
- `approval` (Phase 4)
- `observability` (Phase 5)
- `gateway` (Phase 6)
- `context` (Phase 7)
- `eval` (Phase 8)

See [`docs/architecture/overview.md`](docs/architecture/overview.md) for
what each package is responsible for.
