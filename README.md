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

A single multi-module Maven build (see ADR-0002), one module per
architectural component. Every module is currently an empty skeleton —
see its own `README.md` for its purpose and which phase populates it:

- `payment-domain-service` (Phase 1)
- `agent-runtime` (Phase 2)
- `tool-governance` (Phase 3)
- `human-approval` (Phase 4)
- `observability` (Phase 5)
- `ai-gateway` (Phase 6)
- `enterprise-context` (Phase 7)
- `evaluation` (Phase 8)
