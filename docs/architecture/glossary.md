# Glossary

Terms as used specifically in this project. Where a term has a broader
industry meaning, this defines the sense we mean here.

**Agent** — In the `agent` package: a component that takes a user or
business intent, converses with an LLM, and translates the LLM's output
into one or more tool calls. Not a synonym for "the whole system."

**Tool** — A single, explicitly declared action the agent may request,
with a name, an argument schema, and a policy attached to it in the
`governance` package. If it isn't declared as a tool, the agent cannot do
it.

**Tool Governance** — The mandatory checkpoint (the `governance` package)
between the agent and the payment domain that validates every tool call
against policy before it is allowed to execute. See ADR-0004.

**Human-in-the-loop (HITL) / Human Approval** — The pattern (the
`approval` package) of pausing an agent's action pending explicit human
sign-off, used for actions above a risk threshold.

**Trust boundary** — A line in the architecture where the level of trust
changes and therefore governance controls must be enforced. The most
important one in this project is between the `agent` and `domain`
packages (ADR-0004) - enforced by code structure within one application,
not a network boundary (ADR-0007).

**AI Gateway** — A broker (the `gateway` package) placed in front of the
LLM provider(s) that centralizes credentials, rate limits, cost tracking,
and logging, so no other component talks to a model provider directly.

**Enterprise Context / RAG** — Retrieval-Augmented Generation: supplying
an LLM with relevant text (policy docs, product docs, case history)
retrieved at request time, rather than relying only on what it was
trained on. The `context` package. Deliberately read-only.

**Evaluation (eval)** — Automated scoring of agent behavior against a
fixed set of scenarios/golden answers, run offline (e.g. in CI) to catch
regressions when prompts, models, or tools change. The `eval` package.

**Synthetic domain** — The payment domain modeled in this project is
invented/fictional data and rules, not a copy of any real institution's
systems, so the project can be shared and discussed freely.

**Policy** — A rule enforced by the `governance` package (e.g. "payments
over $10,000 require approval", "only whitelisted currencies allowed").
In this MVP, policy is simple code/config; in enterprise usage this
concept maps to policy-as-code engines (e.g. OPA/Rego).

**ADR (Architecture Decision Record)** — A short, immutable-once-accepted
document recording one architectural decision, its context, and its
consequences. See `docs/adr/` and ADR-0001.

**Logical component** — An architecturally meaningful boundary
(`domain`, `agent`, `governance`, `approval`, `observability`, `gateway`,
`context`, `eval`) implemented as a Java package inside one application,
not a separate Maven module or service, unless and until ADR-0007's
promotion rule is satisfied.

**Identity** — Who the human user is, and separately, who/what the agent
is acting as. Named as an explicit **gap** in this project (ADR-0006) -
not yet owned by any package. Distinct from authorization (`governance`
decides *what* an identity may do; identity itself is *who* is asking).

**Data Classification** — A sensitivity label on a piece of data (e.g.
public / internal / confidential / restricted) that determines what
policy, audit, or retrieval rules should apply to it. Named as an
explicit **gap** in this project (ADR-0006) - not yet owned by any
package.
