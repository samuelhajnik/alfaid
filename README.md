# ALFAID — Assurance Layer For AI Development

ALFAID is a **merge-time understanding gate** for AI-assisted software development.

It is built around one core idea:

> Do not let risky AI-assisted changes pass only because the code looks plausible.  
> Require the developer to prove that they understand the change before merge.

ALFAID is **not** another AI code review assistant.

Its purpose is not to review code for the developer, replace human reviewers, or produce generic pull-request comments. Its purpose is to strengthen developer understanding, verify that the author understands the change, and make that understanding visible before risky changes are accepted.

ALFAID uses analysis only to decide what understanding must be demonstrated.

The central object in ALFAID is not a review comment.

The central object is an **understanding challenge**:

```text
This PR changes retry behavior.
Explain why the operation is safe to retry, what happens after retries are exhausted,
and which test proves duplicate execution is avoided.
```

That is the product: not passive analysis, but visible reasoning before merge.

## The problem ALFAID addresses

AI coding tools can make code generation faster than code ownership.

That creates a dangerous review gap: a pull request can look plausible, tests can appear sufficient, and reviewers can move quickly, while the author still does not fully understand the behavior they are merging.

ALFAID exists to close that review gap.

For review-sensitive changes, ALFAID treats unexplained code as unowned code.

That does not mean the code is automatically wrong. It means the change should not be accepted only because it looks plausible or passes shallow checks. For risky changes, the author should be able to explain the behavior, assumptions, relevant risks, and evidence before the change is merged.

## What ALFAID does

When a PR-like change touches a risky or review-sensitive area, ALFAID turns that change into a challenge-response workflow.

It should:

1. inspect the change enough to identify what must be understood
2. generate a small number of targeted questions for the author
3. require the author to answer with reasoning and evidence
4. evaluate whether the answers demonstrate credible understanding
5. return a decision that reviewers or CI policy can use

The important distinction is this:

```text
Generic review bot:
"This PR may be risky."

ALFAID:
"This PR changes retry behavior. What makes the operation safe to retry, and which test proves duplicate execution is avoided?"
```

ALFAID does not only ask whether the code looks acceptable.

It asks whether the author can explain the change well enough to merge it responsibly.

## Developer experience principle

ALFAID should not make developers' work harder for the sake of process.

The product principle is **targeted friction**:

```text
No challenge for obvious low-risk changes.
Light challenge for moderate-risk changes.
Strong challenge only for changes where shallow understanding would create real merge risk.
```

A useful understanding gate should feel like pre-review preparation, not interrogation.

Developers should benefit from ALFAID when it:

- only triggers on risky or review-sensitive changes
- asks 1–3 relevant questions, not a generic questionnaire
- helps the author prepare a stronger PR explanation
- catches questions a strong reviewer would probably ask anyway
- makes review faster because important context is written down early
- protects the author from merging code they do not fully own yet
- gives a clear path to pass by explaining the change and referencing evidence

The goal is not to create paperwork.

The goal is to make the most important reasoning visible before merge.

## Team and leadership value

For teams adopting AI-assisted development, ALFAID provides a lightweight assurance mechanism.

It does not ban AI tools and it does not slow every pull request. It focuses attention where risk exists.

For engineering leaders, the value is:

- fewer shallow AI-generated changes accepted on plausibility alone
- better review evidence for risky changes
- lower production and maintenance risk
- stronger ownership of merged code
- clearer audit trail for why a risky change was accepted
- less dependence on reviewer intuition alone
- safer AI adoption without blocking developer productivity

ALFAID makes AI-assisted development safer by requiring visible ownership where it matters most.

## Core workflow

```text
PR created or updated
        ↓
ALFAID inspects the change
        ↓
ALFAID identifies what the author must understand
        ↓
ALFAID generates required understanding challenges
        ↓
Author answers with explanation and evidence
        ↓
ALFAID evaluates whether the answers are credible
        ↓
ALFAID returns a decision:
pass / pass_with_warnings / review_required / fail
        ↓
Human reviewer and/or CI policy uses that result
```

The goal is not passive analysis.

The goal is to surface useful developer reasoning before risky merges.

## V1 scope

ALFAID V1 is intentionally focused.

It demonstrates the core product loop:

```text
risky change detected
        ↓
targeted understanding challenge generated
        ↓
author answer required
        ↓
answer evaluated
        ↓
merge decision produced
```

V1 includes:

- PR-like input handling
- deterministic demo scenarios
- challenge-response assurance flow
- structured questions, answers, findings, and decisions
- modular execution through a shared module contract
- two initial assurance modules: **ExplainGate** and **DependencyTrust**

The goal of V1 is not full GitHub/GitLab integration, complete static analysis, or broad security coverage.

The goal of V1 is to prove the product concept clearly:

> Risky AI-assisted changes should require demonstrated understanding before merge.

## V1 modules

### ExplainGate

ExplainGate evaluates whether a risky code change has enough visible reasoning to proceed.

It is based on a simple engineering principle:

> The higher the impact of a change, the more clearly its assumptions and evidence should be stated.

ExplainGate should not create friction for routine edits. It should trigger when the diff changes behavior in areas where shallow understanding can create real merge risk: authorization, retries, persistence, concurrency, error handling, public APIs, data migrations, or production configuration.

For those changes, ExplainGate asks a small number of targeted prompts:

- What assumption makes this change safe?
- What edge case or unsafe assumption matters most?
- What evidence proves the expected behavior?

The exact question depends on the change. The output should help reviewers decide whether the author understands the change well enough to merge it responsibly.

ExplainGate is not a reporting module.

It is the understanding gate.

### DependencyTrust

DependencyTrust evaluates whether a newly introduced dependency is intentional and justified.

It does not attempt complete supply-chain assurance in V1. Instead, it focuses on the first review question that should always be answered:

> Why should this project take on this dependency?

A dependency may be flagged when it is newly added, weakly connected to the diff, redundant with existing functionality, suspiciously named, or not explained by the pull request.

The expected result is not a security verdict.

The expected result is a clear dependency justification or a request for human review.

## Planned next module

### ArchitectureGuard

ArchitectureGuard is planned for V1.5 / V2.

Its purpose is to detect whether changes drift away from the repository's intended architecture, reliability patterns, or documented design claims.

It is deliberately deferred so V1 can stay focused on the two highest-priority assurance questions:

- does the author understand the risky change?
- is the new dependency justified and trustworthy?

## What ALFAID is not

ALFAID is not:

- an AI detector
- a replacement for human code review
- a generic AI code review assistant
- a full static analysis platform
- a complete security auditing system
- a full software supply-chain security product
- a tool that tries to solve every AI-development risk in one release

ALFAID uses analysis only to decide what understanding must be demonstrated.

Its purpose is narrower and stricter:

**Make risky AI-assisted changes harder to merge unless the developer can explain, justify, and support them with evidence.**

## Why this repository exists

This repository is a portfolio project built around a real software-engineering problem:

**AI can make code generation easier than code ownership.**

ALFAID explores one practical response:

**Turn risky pull requests into understanding challenges that must be answered before merge.**

That is the product.

Not generic PR review.

Not AI authorship detection.

Not automated reviewer replacement.

## Status

Design-first project.

The current source of truth for V1 is [`DESIGN.md`](./DESIGN.md).

Current phase:

- product concept defined
- V1 module scope defined
- source-of-truth design defined
- implementation not yet started

## Principles

- strengthen human review rather than replace it
- optimize for demonstrated understanding, not authorship detection
- apply friction only where risk justifies it
- ask targeted questions, not generic questionnaires
- keep assurance outputs evidence-based and inspectable
- prefer a focused product over a vague AI platform
- preserve modular boundaries so assurance concerns can evolve over time
- build thin vertical slices and validate them against the design

## Repository roadmap

```text
V1
├── ExplainGate
├── DependencyTrust
├── challenge-response assurance flow
├── shared module contract
├── deterministic PR-like fixtures
└── structured decision output

V1.5 / V2
└── ArchitectureGuard

Later
└── additional assurance modules as requirements evolve
```

## Intended reviewer takeaway

ALFAID is not another generic AI coding demo or PR review bot.

It is a focused attempt to solve a specific modern problem:

**Teams can accept AI-assisted changes faster than they can truly understand them.**

ALFAID treats that as an assurance problem and turns pull request review into something stricter, but still practical:

**Risky changes should require targeted explanation, evidence, and visible ownership before merge.**
