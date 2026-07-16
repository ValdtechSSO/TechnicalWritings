# AOSE in Brief: Stop Trusting Your Agent. Start Verifying It.

*A short introduction to Agent-Oriented Software Engineering. The full RFC will be available soon.*

---

A few months ago, an AI agent handed me an implementation report for a core library I'd asked it to extract and package. The report said: **complete**. Tests passing. Ready to merge.

It wasn't. When I reviewed it myself, I found missing test diffs, no regression comparison against the previous behavior, and code that had never been audited. The report was fluent, confident, well-structured — and wrong. If I had trusted it, I would have shipped a broken foundation into two products at once.

That failure wasn't the model's fault. It was mine. I had built a workflow where "done" was something the agent *said*, not something the system *proved*.

This is the central problem of agent-assisted engineering in 2026, and it's why I wrote an RFC proposing an interpretation of **Agent-Oriented Software Engineering (AOSE)** for this new context. Not "how to build software using agents" — everyone is already doing that. This proposal is about how to design your repositories, workflows, and controls so that agent work can be *trusted*, at increasing levels of autonomy, without you becoming the bottleneck or the victim.

## A note on the term

I did not coin AOSE. The term comes from an established research tradition, active since at least the AOSE 2000 workshop, concerned with analyzing, designing, and implementing software systems whose runtime components are autonomous or semi-autonomous agents.

This RFC takes that existing term and adapts its meaning to a related problem: how to engineer software-development environments in which humans collaborate with AI coding agents. It does not replace or claim ownership of classical AOSE; it applies the term's agent-oriented perspective to the context, permissions, contracts, workflows, and evidence that make agent-assisted engineering trustworthy.

Here's the whole methodology in four claims.

---

## Claim 1: The primary artifact is no longer just source code

Code produced by an agent cannot be evaluated on its own. To judge it, you need to know: what context the agent received, what permissions it held, what contracts it had to satisfy, what checks it ran, and what evidence it produced.

That means the *development context* — instruction files, workflows, skills, policies, risk classifications, evidence templates — is now a first-class engineering artifact. It lives in the repo. It's versioned. It's reviewed. When an agent fails, you don't just fix the code; you fix the context that let the failure happen.

Most repositories today are optimized for human memory and tribal knowledge. Instructions are scattered across READMEs, wikis, chat threads, and the heads of senior engineers. An agent can be perfectly capable of writing correct code and still fail because it never discovered the one command, invariant, or constraint that every maintainer knows by heart.

The fix is not "more context." Dumping everything into a giant AGENTS.md measurably *hurts* task success. The fix is **minimal sufficient context**: a small router file that sends the agent to exactly the scoped documents, skills, and rules it needs for *this* task, and nothing else.

## Claim 2: The model proposes. Deterministic systems decide.

Language models are excellent at semantic judgment: interpreting ambiguous requests, planning, classifying failures, spotting suspicious patterns. They are terrible authorities on facts that are machine-checkable: whether the build passed, whether a migration is reversible, whether a deployment is authorized.

So split the system in two:

- **Reasoning plane** (probabilistic): the model interprets, plans, generates, reviews.
- **Control plane** (deterministic): schemas validate, tests execute, policies authorize, gates block.

The execution pattern is always: **Propose → Validate → Authorize → Execute → Observe → Prove.**

A model output must never directly cause a sensitive side effect. If your prompt says "never deploy without approval" but the agent holds deployment credentials, you don't have a policy — you have a suggestion. Enforcement belongs at the tool boundary, in code, where it cannot be talked out of.

I learned this pattern building a conversational assistant for a sports-management platform. The LLM resolves "show me Adrian's red cards this season" into a typed, structured proposal — intent, player ID, season ID, event types. Then deterministic code validates the schema, checks that the IDs exist, verifies authorization, and executes a known query. The model interprets language. It never invents SQL, never bypasses auth, never decides that an invalid ID is "probably fine." Tool-selection accuracy went from 65% to 95% with the same model, purely from this separation.

## Claim 3: One good agent beats a committee of mediocre ones

The dominant aesthetic right now is the multi-agent pipeline: Planner → Architect → Backend → Frontend → QA → Reviewer. It looks like an org chart, and that's exactly the problem. Every handoff loses information. Every specialist re-reads the same context. Cost and latency multiply, and nobody owns the final synthesis.

AOSE's default is **one capable agent with good tools and minimal context**. Multi-agent topologies are legitimate — planner-executor when a plan needs approval before execution, cross-model review for high-risk changes, parallel workers for genuinely independent tasks — but each one must be justified by a measured failure of the simpler design, not adopted as a maturity badge.

The same discipline applies to knowledge infrastructure. Start with structured Markdown and grep. Add a generated index when search fails. Add semantic retrieval when *measured* retrieval failures justify it. Add a knowledge graph almost never. Building a vector database before your documents have owners and sources of truth just means retrieving stale contradictions faster.

**Agent count is an architectural variable, not a maturity badge.** So is your retrieval stack.

## Claim 4: Completion is a claim that requires evidence

This is the claim my merge-blocker incident taught me, and it's the heart of the methodology.

Agents are fluent even when wrong. "All tests pass," "this change is safe," "the implementation is complete" — these are confidence statements, and confidence is the *weakest* form of evidence. AOSE replaces them with verifiable artifacts: the command that ran, the environment it ran in, the exit code, the test counts, the benchmark baseline and result, the checks that were *skipped* and why.

Concretely, every non-trivial task produces an **evidence manifest** — a machine-readable record where completion status is *derived* from policy and checks, never set because the agent asserted it. If the concurrency test wasn't implemented, the manifest says `"status": "not-run"` and `"completionClaim": false`, no matter how confident the summary sounds.

The rule is four words: **no evidence, no completion.**

This pairs with explicit **trust boundaries**. Autonomy isn't binary; it's a function of risk. An agent renaming a variable on an isolated branch needs almost no supervision. An agent touching authentication, payments, or destructive migrations needs strong deterministic validation plus accountable human approval. The allowed autonomy level is a policy decision written down in the repo — never an assumption the agent makes about itself.

---

## Where to start (without building a platform)

You don't need an agent orchestration framework to adopt this. The minimum viable version is:

1. A concise `AGENTS.md` that routes instead of lecturing.
2. One standard workflow for features and bug fixes.
3. One authoritative validation command.
4. A simple risk/approval matrix: what the agent may do alone, what needs review, what's forbidden.
5. A task worksheet template — so an interrupted task can be picked up by another agent (or by you) without archaeology.
6. An evidence template, and the rule that completion requires it.
7. A habit: every recurring failure becomes a test, a tool, a policy, or a doc fix.

That's a weekend of setup, and it delivers most of the value before you ever consider multi-agent orchestration or semantic retrieval.

## What this is not

AOSE is not a claim that agents replace engineers, not a mandate to run multiple agents, not a vector-database requirement, and definitely not a license to auto-deploy to production. It's the opposite of both extremes in the current debate: neither "agents can't be trusted" nor "let them run unsupervised."

The future isn't software built by unsupervised agents. The future is software engineering where agents earn increasing autonomy *because* their work is bounded by contracts, controlled by policy, observed through traces, and proven through evidence.

The full RFC covers the parts this post skips: the control model, context assembly, agent topologies and when each is justified, testing strategy including false-confidence audits, security and prompt-injection surfaces, a maturity model for progressive adoption, and a catalogue of anti-patterns ("Multi-Agent Theatre," "Done Without Execution," "LLM as Policy Engine").

**Read the spec here: [link]. It's a draft RFC — v0.2 — and the discussion is open. Tell me where it's wrong.**

---

*Adrian Mustelier builds production systems where LLM judgment meets deterministic enforcement: a grassroots football platform, a legal-tech SaaS, and a knowledge-management engine — mostly solo, mostly with agents doing the typing.*
