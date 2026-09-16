# Glossary

Terms as used specifically in this project. Where a term has a broader
industry meaning, this defines the sense we mean here.

**Agent** — In `agent-runtime`: a component that takes a user or business
intent, converses with an LLM, and translates the LLM's output into one or
more tool calls. Not a synonym for "the whole system."

**Tool** — A single, explicitly declared action the agent may request,
with a name, an argument schema, and a policy attached to it in
`tool-governance`. If it isn't declared as a tool, the agent cannot do it.

**Tool Governance** — The mandatory checkpoint (module: `tool-governance`)
between the agent and the payment domain that validates every tool call
against policy before it is allowed to execute. See ADR-0004.

**Human-in-the-loop (HITL) / Human Approval** — The pattern (module:
`human-approval`) of pausing an agent's action pending explicit human
sign-off, used for actions above a risk threshold.

**Trust boundary** — A line in the architecture where the level of trust
changes and therefore governance controls must be enforced. The most
important one in this project is between `agent-runtime` and
`payment-domain-service` (ADR-0004).

**AI Gateway** — A broker (module: `ai-gateway`) placed in front of the
LLM provider(s) that centralizes credentials, rate limits, cost tracking,
and logging, so no other component talks to a model provider directly.

**Enterprise Context / RAG** — Retrieval-Augmented Generation: supplying
an LLM with relevant text (policy docs, product docs, case history)
retrieved at request time, rather than relying only on what it was
trained on. Module: `enterprise-context`. Deliberately read-only.

**Evaluation (eval)** — Automated scoring of agent behavior against a
fixed set of scenarios/golden answers, run offline (e.g. in CI) to catch
regressions when prompts, models, or tools change. Module: `evaluation`.

**Synthetic domain** — The payment domain modeled in this project is
invented/fictional data and rules, not a copy of any real institution's
systems, so the project can be shared and discussed freely.

**Policy** — A rule enforced by `tool-governance` (e.g. "payments over
$10,000 require approval", "only whitelisted currencies allowed"). In
this MVP, policy is simple code/config; in enterprise usage this concept
maps to policy-as-code engines (e.g. OPA/Rego).

**ADR (Architecture Decision Record)** — A short, immutable-once-accepted
document recording one architectural decision, its context, and its
consequences. See `docs/adr/` and ADR-0001.
