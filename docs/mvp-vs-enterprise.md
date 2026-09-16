# MVP vs. Enterprise — the running ledger

This document tracks, per component, what this learning project does for
the MVP and what a real enterprise deployment would need instead, so the
gap is always visible rather than implied. Update it whenever a phase
lands and makes a simplification.

| Component | This project (MVP) | A real enterprise | Why we simplify here |
|---|---|---|---|
| Payment domain | One Spring Boot service covering payment, fraud/risk, authorization, settlement, and incident concepts as small sub-packages, synthetic data, in-process rules | Many systems (core banking, ledger, fraud, sanctions screening), each with its own team and SLAs | The point is to demonstrate agent *governance*, not to rebuild core banking; one realistic service with shallow-but-coherent sub-areas is enough to have real invariants and a genuine investigation surface |
| Repository/build topology | Single repo, single Maven module / single Spring Boot app; components are packages (ADR-0007, superseding ADR-0002) | Separate repos per component, independently deployed, versioned contracts between them | A module or service split is only worth its ceremony once a component has a concrete reason to be independently built/deployed/owned - none does yet |
| Agent ↔ domain boundary | Enforced by package structure + code review; an architecture test (ArchUnit) added once real code exists (ADR-0004, ADR-0007) | Enforced by network policy/service mesh + mTLS so it's physically unbypassable | No infra to segment networks in a single-process learning app; the *architectural intent* is still real and testable |
| Tool governance | Simple rule table / interceptor in code | Policy-as-code engine (e.g. OPA/Rego), centrally managed, audited, versioned separately from application code | A hand-rolled policy layer is enough to demonstrate the concept; a real policy engine is a separate, mature discipline |
| Human approval | Approval queue table + REST endpoint, single approver | Full workflow engine (e.g. Camunda/Temporal), maker-checker with role-based multi-party approval, SLA escalation | Durable pause/resume is the concept being taught; multi-party workflow adds process complexity, not architectural insight |
| AI gateway | Thin proxy/logging interceptor, maybe backed by Spring AI's own abstractions | Dedicated gateway product (e.g. LiteLLM, Kong AI Gateway, Portkey) with multi-tenant cost allocation, model allowlists | Demonstrates *why* the boundary exists; a production gateway is a build-vs-buy decision most enterprises would buy; Phase 6 evaluates 2-3 realistic options rather than the full enterprise vendor landscape (Azure AI Foundry, Bedrock, Vertex AI, Kong, etc.), which isn't practical to trial hands-on in a learning project |
| Enterprise context / RAG | Small local document set, simple vector store | Enterprise search integration, access-control-aware retrieval (a user/agent only retrieves what its own permissions allow), document freshness pipelines | The governance lesson (keep retrieval read-only, separate from action) doesn't require enterprise-scale corpora |
| Observability | Structured logging, maybe a local trace exporter | OpenTelemetry GenAI semantic conventions feeding a real trace store (e.g. Tempo/Jaeger) with retention/redaction policy | Enough to see the shape of "AI observability ≠ APM"; starts with Actuator + structured logs before introducing the full OTel/Prometheus/Grafana stack, so infra weight is added only once there's real behavior worth observing |
| Evaluation | A handful of golden scenarios (normal, ambiguous, out-of-policy, adversarial) run manually/in CI | Continuous, statistically significant eval sets, human-graded samples, drift monitoring in production | Small golden sets are enough to demonstrate the CI-gate pattern; production-scale eval is its own specialty (and cost center) |
| Identity *(gap - ADR-0006)* | Not modeled yet; no distinct user vs. agent identity | Enterprise IdP/IAM (e.g. OIDC), separate credentials/scopes for the agent's own service identity vs. the human it acts on behalf of | Deferred until a phase actually needs to distinguish "who asked" from "what's allowed" - resolved starting Phase 3 (agent identity + basic RBAC) |
| Data classification *(gap - ADR-0006)* | Not modeled yet; all synthetic data treated the same | A labeling scheme (public/internal/confidential/restricted) driving policy, audit, and retrieval rules | Deferred until policy or retrieval needs to treat data differently by sensitivity - likely Phase 3 or Phase 7 |
| Threat modeling | Lightweight, manually maintained threat→phase map (`docs/security/threat-model.md`) | Formal threat-modeling process (e.g. STRIDE workshops), red-team exercises, continuous threat intel | A named, versioned lightweight model keeps risks visible; a full program is disproportionate for a learning project |

## How to use this document

Each phase's closing commit should add or update a row here if it
introduces a new simplification, so this stays the single place to answer
"what did we skip, and what would it take to not skip it."
