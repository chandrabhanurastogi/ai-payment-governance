# Enterprise Context / RAG

**Phase:** 7 — Enterprise context / RAG
**Status:** empty skeleton (no source code yet)

## Purpose

Retrieval-augmented context: policy documents, product documentation, and
historical case data the agent can read to inform its decisions.

## Why this is a separate module, and read-only

This is a knowledge plane, deliberately kept apart from
`payment-domain-service` (the transactional plane) and reached only for
reads. Conflating "things the agent can read for context" with "things
the agent can act on" widens the blast radius of prompt injection and
data-exfiltration risks - keeping the boundary explicit is itself one of
the governance lessons this project demonstrates.

## What it will NOT contain

Any write path back into the payment domain, and no tool that lets
retrieved content trigger a domain action without passing back through
`tool-governance`.
