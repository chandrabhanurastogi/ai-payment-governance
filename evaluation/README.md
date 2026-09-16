# Evaluation

**Phase:** 8 — AI evaluation
**Status:** empty skeleton (no source code yet)

## Purpose

A harness that runs the agent against golden datasets/scenarios and scores
its behavior (correctness, policy compliance, safety) before and after
changes to prompts, models, or tools.

## Why this is a separate module, outside the runtime path

Evaluation is a CI/pipeline concern, not something the running system
depends on. It is modeled as its own module because it should be possible
to run evaluations against `agent-runtime` from the outside (as a
consumer), the same way integration tests exercise a service without
being part of it.

## What it will NOT contain

Any code the running agent depends on at request time - a bug in
`evaluation` should never be able to affect production behavior.
