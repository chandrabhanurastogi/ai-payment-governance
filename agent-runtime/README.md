# Agent Runtime

**Phase:** 2 — First AI agent
**Status:** empty skeleton (no source code yet)

## Purpose

Hosts the Spring AI agent: prompt construction, conversation/session state,
and the loop that turns a user or business request into one or more tool
calls.

## Why this is a separate module

This module is a distinct trust domain from `payment-domain-service`. It
must never hold a database connection to the payment domain and must never
call it directly - all business actions are reached through
`tool-governance`. Enforcing that as a module boundary now (even before
any enforcement is possible at runtime) makes the boundary visible in code
reviews from the first commit. See ADR-0004.

## What it will NOT contain

Direct calls to the payment domain's persistence layer or REST API. Any
tool the agent invokes is declared and mediated by `tool-governance`.
