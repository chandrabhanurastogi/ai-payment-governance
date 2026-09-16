# 0003. Java, Spring Boot, and Spring AI as the primary stack

Date: 2026-09-16

## Status

Accepted

## Context

The project's stated goal is to demonstrate *enterprise* AI agent
governance patterns, not to build the fastest possible prototype. The
stack should be one a large, JVM-centric enterprise would plausibly
already be running, and should give first-class support for the
capabilities the later phases need: tool/function calling, RAG, and
(increasingly) MCP-style tool protocols.

## Decision

We will build on **Java 21 (LTS), Spring Boot, and Spring AI** as the
primary application stack across all modules that become services.

## Consequences

**For this learning MVP:** Spring AI provides ready-made abstractions for
chat models, structured tool/function calling, and vector-store-backed
RAG - directly relevant to Phases 2, 3, and 7 - so we're not hand-rolling
provider SDK integration. Spring Boot's ecosystem (Actuator, Micrometer,
its OpenTelemetry bridge) gives Phase 5 observability a running start
without extra plumbing. Using an LTS Java version keeps us on a stable
language/runtime baseline for the life of the project.

**For a real enterprise:** this is a realistic choice - Java/Spring is
still the dominant stack in banking and payments back ends - but a real
adoption would also weigh organizational factors we're skipping here:
existing platform team standards, whether a polyglot approach (e.g. a
Python-based eval/RAG sidecar next to a Java core) better fits available
talent, and vendor support agreements for the AI SDK in use.

**Risk accepted:** Spring AI is younger and faster-moving than the
broader LLM-tooling ecosystem (e.g. LangChain's Python/JS libraries). Its
API surface may change between phases; we will pin explicit versions in a
follow-up ADR when Phase 2 introduces the first dependency, rather than
floating a version range.

## Notes

Alternative considered: Python (LangChain/LlamaIndex) for the AI-facing
modules only, Java for the payment domain. Rejected for now to keep a
single toolchain while learning; revisiting this as a polyglot ADR is
possible in a later phase if a real limitation of Spring AI is hit.
