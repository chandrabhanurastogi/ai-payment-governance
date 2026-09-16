# Tool Governance

**Phase:** 3 — Tool governance
**Status:** empty skeleton (no source code yet)

## Purpose

The mandatory interception point between `agent-runtime` and
`payment-domain-service`. Declares which tools exist, their schemas, and
the policy checks (limits, allowed currencies, allowed operations) that
must pass before a tool call is allowed to reach the domain.

## Why this is the most important module in the repository

This is the boundary that makes "an AI acting on payments" governable at
all. Without it, "AI governance" is just a slogan attached to a system
that lets a model call anything it wants. See ADR-0004 and
`docs/architecture/overview.md`.

## What it will NOT contain

Agent/LLM-specific code (prompting, model selection) or payment business
logic. It only knows about tool contracts and policy - it should be
possible to unit test its policy decisions with no model and no database
in the loop.
