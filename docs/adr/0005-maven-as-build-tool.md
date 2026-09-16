# 0005. Maven as the build tool

Date: 2026-09-16

## Status

Accepted

## Context

A multi-module repository (ADR-0002) needs one build tool chosen up
front, since it determines the repo skeleton (POMs vs. Gradle build
scripts) created in Phase 0.

## Decision

We will use **Maven**, with one root `pom.xml` (`packaging=pom`) listing
all modules, and one `pom.xml` per module inheriting from it.

## Consequences

**For this learning MVP:** Maven's declarative XML keeps `dependencyManagement`
centralized in the root POM, which matters here because Spring AI's
dependency versions are still moving relatively quickly across modules -
centralizing them avoids version drift between modules. Maven is also the
most ubiquitous choice for Spring Boot tutorials/documentation, which
matters for a learning project where being able to cross-reference
official docs matters.

**For a real enterprise:** Gradle is common too, especially where build
performance (incremental/parallel builds, build caching) matters at scale
across many modules, or where custom build logic is needed. Once this
project has more than a handful of modules with real dependencies, the
verbosity trade-off Maven makes could become a real cost - that would be
a legitimate reason to revisit this ADR later, not a reason to avoid
deciding now.

## Notes

Alternative considered: Gradle. Rejected for now on familiarity/verbosity
trade-offs; not rejected as "wrong", just not the default for this
project.
