# Human Approval

**Phase:** 4 — Human approval
**Status:** empty skeleton (no source code yet)

## Purpose

Holds an agent action in a durable "pending" state when `tool-governance`
flags it as requiring human sign-off, and resumes execution once a human
approves or rejects it.

## Why this is a separate module

Approval state must survive process restarts - an agent action can be
"awaiting approval" for hours or days. That durability requirement is
different in kind from the request/response flow in `agent-runtime`,
which is why it is modeled as its own component rather than an in-memory
callback inside the agent loop.

## What it will NOT contain

The policy decision of *whether* something needs approval - that is
`tool-governance`'s job. This module only implements the pause/resume
mechanics and the approval record.
