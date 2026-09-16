# AI Payment Governance Learning Project

## Purpose

This is a learning/reference project exploring how an enterprise with thousands of engineers could safely use AI agents in software engineering and payment operations.

The goal is to understand the architecture, governance, tooling, context management, security and observability required to scale AI responsibly.

This is not a production payment system.

All payment data must be synthetic.

## Technology

Primary technology:

* Java 21+
* Spring Boot
* Spring AI
* Spring Security
* PostgreSQL
* Maven
* Docker
* Testcontainers
* OpenTelemetry
* Prometheus
* Grafana

Prefer simple, understandable technologies over unnecessary infrastructure.

## Engineering Principles

1. Explain architectural decisions before implementing them.
2. Prefer the simplest architecture that demonstrates the capability.
3. Do not create microservices merely to make the architecture look impressive.
4. Separate domain functionality from AI functionality and governance.
5. Agents must never have unrestricted database access.
6. Agents interact with domain systems through explicit tools.
7. Sensitive operations require authorization and, where appropriate, human approval.
8. Use synthetic payment data only.
9. Never put secrets or credentials into source code.
10. Every significant architectural decision should have an ADR.
11. Add tests with each meaningful implementation.
12. Keep documentation synchronized with implementation.

## Learning Mode

This project is primarily for learning.

When proposing an architectural component, explain:

* What problem it solves
* Why we need it
* What alternatives exist
* Why the proposed solution was selected
* Whether it is required for the MVP
* What an enterprise implementation might look like

Clearly distinguish:

MVP

from

Future enterprise architecture.

Do not hide complexity behind abstractions without explaining it.

## AI Agent Principles

An agent consists of:

* reasoning/model
* instructions
* context
* tools
* policies
* state
* observability
* audit

Agents must only access capabilities through explicit tools.

Tools must enforce authorization and validation.

## Governance Principles

The system should eventually demonstrate:

* user identity
* agent identity
* tool authorization
* data classification
* policy evaluation
* audit logging
* human approval
* observability
* AI evaluation

## Git Workflow

Make small, meaningful commits.

Before significant changes:

1. Explain the planned change.
2. Identify affected components.
3. Implement the smallest useful change.
4. Add/update tests.
5. Update documentation.
6. Show the files changed.
7. Suggest an appropriate commit message.

Do not make large unrelated changes in a single commit.

## Current Development Rule

Always respect the current project phase.

Do not implement future phases unless explicitly requested.

When requirements are unclear, explain the options and recommend an approach rather than silently making a major architectural decision.

