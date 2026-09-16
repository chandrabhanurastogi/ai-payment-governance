# 0002. Single multi-module repository

Date: 2026-09-16

## Status

Accepted

## Context

The eventual system has at least eight architecturally distinct
components (payment domain, agent runtime, tool governance, human
approval, observability, AI gateway, enterprise context/RAG, evaluation).
We need to decide, before writing any build files, whether these live as
separate repositories (as they likely would at a real enterprise, owned
by different teams) or as modules within one repository.

## Decision

We will use a **single Git repository with a multi-module Maven build**.
Each architectural component gets its own top-level Maven module
(`payment-domain-service`, `agent-runtime`, `tool-governance`,
`human-approval`, `observability`, `ai-gateway`, `enterprise-context`,
`evaluation`), governed by one root `pom.xml`.

Module boundaries are still treated as if they were repository/service
boundaries: no module reaches into another module's internals, and
inter-module calls happen only through the interfaces defined in
`docs/architecture/overview.md` (e.g. agent-runtime -> tool-governance,
never agent-runtime -> payment-domain-service). This means the modules
could be split into separate repositories later without redesigning the
boundaries - only the build/deploy topology would change.

## Consequences

**For this learning MVP:** one place to browse the whole system's
evolution, a single CI pipeline, and the ability to make atomic
cross-module commits while still learning what the right boundaries are.
Lower ceremony than managing eight repos solo.

**For a real enterprise:** each of these components would very likely be
owned by a different team, deployed independently, and versioned against
the others via explicit contracts (OpenAPI specs, event schemas, a shared
client SDK) rather than direct Java method calls - because independent
deployability and independent ownership are the actual goals, not just
code organization. A monorepo can still work at enterprise scale (many
large companies do this), but it requires investment we're not making
here: build caching/selective builds, code ownership files, and
per-module release trains.

## Notes

Alternative considered: one repository per component from the start.
Rejected for this phase because it multiplies setup overhead (eight
CI configs, eight READMEs to keep in sync) before there is any code to
justify the separation, and because a learning project benefits from
being able to see the whole system in one `git log`.
