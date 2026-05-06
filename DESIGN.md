# ALFAID Design — Assurance Layer For AI Development

This document is the source of truth for the V1 product and technical design.

ALFAID is a merge-time understanding gate for AI-assisted software development. Its purpose is to make risky pull requests harder to merge unless the author can demonstrate that they understand the change, its assumptions, its relevant risks, and the evidence supporting it.

ALFAID is not intended to be a generic AI code review assistant. The product is built around a stricter idea:

> Do not let risky AI-assisted changes pass only because the code looks plausible.  
> Require the developer to prove that they understand the change before merge.

## 1. Product intent

AI coding tools can produce plausible code quickly. That speed creates a review gap: developers and reviewers may accept code before anyone has built enough understanding of its behavior, risks, and long-term ownership cost.

ALFAID closes that review gap by adding a challenge-response assurance step to pull request review.

The system should:

- detect changes that deserve stronger explanation
- generate a small number of targeted questions for the author
- require concrete answers and evidence
- evaluate whether the answers are credible
- return a decision that can guide human review or CI policy

The product is not optimized for detecting whether code was written by AI. It is optimized for verifying whether the author understands the change well enough to merge it responsibly.

### 1.1 Developer experience principle: targeted friction

ALFAID should not make developers' work harder for the sake of process.

The product principle is **targeted friction**:

```text
No challenge for obvious low-risk changes.
Light challenge for moderate-risk changes.
Strong challenge only for changes where shallow understanding would create real merge risk.
```

A tool that challenges every pull request will be ignored. A useful assurance layer applies friction only where lack of demonstrated understanding would meaningfully increase merge risk.

In V1, this means:

- low-risk changes should usually pass without an author challenge
- risky changes should receive a small number of relevant questions
- questions should be specific to the change, not generic checklists
- answers should be evaluated for specificity, relevance, and evidence
- the author should have a clear path to pass the gate

The intended author experience is closer to pre-review preparation than interrogation. ALFAID should surface the questions a strong reviewer is likely to ask anyway, while the context is still fresh.

### 1.2 Leadership and team value

For teams adopting AI-assisted development, ALFAID provides a lightweight assurance mechanism.

It does not ban AI tools and it does not slow every pull request. It focuses attention where risk exists.

The value for teams is:

- fewer shallow AI-generated changes accepted on plausibility alone
- better review evidence for risky changes
- lower production and maintenance risk
- stronger ownership of merged code
- clearer audit trail for why a risky change was accepted
- less dependence on reviewer intuition alone
- safer AI adoption without blocking developer productivity

## 2. Core workflow

```text
PR-like input is created or updated
        ↓
ALFAID builds a change context
        ↓
Enabled modules analyze the change
        ↓
Modules emit findings and targeted author questions
        ↓
Author submits answers with reasoning and evidence
        ↓
ALFAID evaluates the answers
        ↓
ALFAID aggregates module results
        ↓
ALFAID returns a decision:
pass / pass_with_warnings / review_required / fail
```

The workflow is intentionally interactive. A finding alone is not the product. The product is the requirement that the author responds to risk with explanation and evidence before merge.

## 3. V1 scope

V1 demonstrates the core assurance loop using deterministic PR-like fixtures.

V1 includes:

- PR-like input model
- changed-file and diff summary representation
- module execution pipeline
- challenge-response lifecycle
- structured findings
- structured author questions
- structured author answers
- answer evaluation
- decision aggregation
- CLI-oriented demo flow
- deterministic fixtures for repeatable review
- two modules: ExplainGate and DependencyTrust

V1 does not include:

- full GitHub/GitLab integration
- complete static analysis
- complete security auditing
- advanced repository-wide semantic analysis
- production-grade policy management
- organization-specific rule configuration
- automatic merge blocking in a real hosted platform

The V1 implementation should prove the concept, not simulate an entire enterprise review platform.

## 4. System boundaries

ALFAID operates on PR-like input.

A V1 input should contain enough information to simulate a pull request review:

- PR identifier
- title and description
- changed files
- diff snippets or change summaries
- dependency manifest changes, where relevant
- optional test evidence
- optional repository context
- author answers, once available

V1 should be runnable locally through deterministic demo scenarios. Real provider integration can come later.

## 5. High-level architecture

```text
+-------------------+
| PR-like fixture   |
+---------+---------+
          |
          v
+-------------------+
| Change Context    |
| Builder           |
+---------+---------+
          |
          v
+-------------------+        +-------------------+
| Module Runner     +------->| ExplainGate       |
+---------+---------+        +-------------------+
          |                  +-------------------+
          +----------------->| DependencyTrust   |
          |                  +-------------------+
          v
+-------------------+
| Challenge Set     |
+---------+---------+
          |
          v
+-------------------+
| Author Answers    |
+---------+---------+
          |
          v
+-------------------+
| Answer Evaluator  |
+---------+---------+
          |
          v
+-------------------+
| Decision Engine   |
+---------+---------+
          |
          v
+-------------------+
| Report Output     |
| JSON / Markdown   |
+-------------------+
```

The architecture should keep modules independent. Each module receives a shared change context and emits findings, questions, and evaluation results through a common contract.

## 6. Domain model

The implementation language can determine the final types, but V1 should preserve these concepts.

### ChangeContext

Represents the PR-like change being evaluated.

```text
ChangeContext
├── pr_id
├── title
├── description
├── files_changed
├── diff_summaries
├── dependency_changes
├── test_evidence
└── repository_context
```

### Finding

Represents something assurance-relevant discovered by a module.

```text
Finding
├── id
├── module
├── severity
├── category
├── file_refs
├── summary
├── rationale
└── required_questions
```

Severity should remain simple in V1:

```text
low / medium / high
```

### Question

Represents a targeted author response required by a module.

```text
Question
├── id
├── finding_id
├── module
├── prompt
├── expected_evidence
├── risk_context
└── required
```

A question should be specific enough that a shallow generic answer is visibly insufficient.

### AuthorAnswer

Represents the author's response to a question.

```text
AuthorAnswer
├── question_id
├── answer_text
├── evidence_refs
└── submitted_by
```

### EvaluationResult

Represents the assessment of an author's answer.

```text
EvaluationResult
├── question_id
├── status
├── confidence
├── rationale
└── missing_evidence
```

Possible statuses:

```text
credible / weak / missing / contradicted
```

### Decision

Represents the final ALFAID result.

```text
Decision
├── status
├── summary
├── module_results
├── blocking_findings
├── warnings
└── recommended_next_actions
```

Possible statuses:

```text
pass / pass_with_warnings / review_required / fail
```

## 7. Module contract

Each assurance module should follow the same lifecycle.

```text
Module receives ChangeContext
        ↓
Module analyzes relevant parts of the change
        ↓
Module emits Findings
        ↓
Module emits targeted Questions for findings that require author response
        ↓
Module evaluates AuthorAnswers for its own Questions
        ↓
Module returns a module-level result
```

A module should not decide the final repository-wide outcome alone unless it emits a blocking result. Final aggregation belongs to the Decision Engine.

### Module interface shape

The implementation can refine naming, but the contract should resemble:

```text
AssuranceModule
├── name
├── analyze(change_context) -> ModuleAnalysis
└── evaluate_answers(change_context, answers) -> ModuleEvaluation
```

### ModuleAnalysis

```text
ModuleAnalysis
├── module
├── findings
├── questions
└── summary
```

### ModuleEvaluation

```text
ModuleEvaluation
├── module
├── evaluation_results
├── module_decision
└── rationale
```

## 8. ExplainGate design

ExplainGate is the primary V1 module.

Its purpose is to evaluate whether a risky code change has enough visible reasoning to proceed.

It is based on a simple engineering principle:

> The higher the impact of a change, the more clearly its assumptions and evidence should be stated.

ExplainGate should not create friction for routine edits. It should trigger when the diff changes behavior in areas where shallow understanding can create real merge risk: authorization, retries, persistence, concurrency, error handling, public APIs, data migrations, or production configuration.

### ExplainGate triggers

In V1, ExplainGate should trigger on simple, inspectable signals rather than complex semantic analysis.

Possible trigger categories:

- behavior-changing code paths
- error handling changes
- retry, timeout, or fallback logic
- persistence or data model changes
- concurrency-sensitive logic
- authentication or authorization logic
- validation changes
- public API behavior changes
- test removal or weakened test coverage
- large changes with unclear PR description

The V1 implementation can use deterministic fixture metadata and lightweight heuristics. It does not need perfect detection.

### ExplainGate question generation

ExplainGate should produce targeted prompts, not generic review comments or a full questionnaire.

Expected V1 behavior:

```text
Low-risk change:
- no challenge

Moderate-risk change:
- 1 targeted question

High-risk change:
- 2-3 targeted questions, focused on the most important assumption, edge case, or evidence gap
```

For risky changes, ExplainGate asks a small number of prompts such as:

- What assumption makes this change safe?
- What edge case or unsafe assumption matters most?
- What evidence proves the expected behavior?

The exact question depends on the change. Good prompts are specific to the diff and help reviewers decide whether the author understands the change well enough to merge it responsibly.

Examples:

```text
This PR changes retry behavior. What assumption makes this operation safe to retry, and which test proves duplicate execution is avoided?
```

```text
This PR changes authorization logic. What prevents a user from accessing data they should not see?
```

```text
This PR changes transaction boundaries. Which operations must commit atomically, and what inconsistent state is possible if they do not?
```

The goal is not to create paperwork. The goal is to make the most important reasoning visible before merge.

### ExplainGate answer evaluation

ExplainGate should evaluate whether the answer is credible, not whether it is beautifully written.

A credible answer should usually include:

- a direct explanation of the change
- a correctness argument
- assumptions or constraints
- at least one relevant edge case, unsafe assumption, or risk, when applicable
- evidence reference, such as a test, fixture, log, or documented reasoning

A weak answer usually has one or more problems:

- repeats the PR title without explaining behavior
- says only that tests pass
- gives no assumptions
- does not identify the relevant risk for a risk-sensitive change
- gives no evidence
- does not answer the specific question

V1 answer evaluation should be deterministic and fixture-driven. A question may define expected concepts, expected evidence references, weak generic phrases, and expected test names. The evaluator should not attempt broad semantic judgment in V1. It should explain why an answer was classified as credible, weak, missing, or contradicted.

## 9. DependencyTrust design

DependencyTrust evaluates whether a newly introduced dependency is intentional and justified.

It does not attempt complete supply-chain assurance in V1. Instead, it focuses on the first review question that should always be answered:

> Why should this project take on this dependency?

Its purpose is to prevent shallow acceptance of generated, unnecessary, redundant, suspicious, unused, or weakly justified package additions.

### DependencyTrust triggers

DependencyTrust should trigger when a dependency manifest changes.

Examples:

- `package.json`
- `package-lock.json`
- `pom.xml`
- `build.gradle`
- `requirements.txt`
- `go.mod`

V1 does not need to support every ecosystem. It can support one or two clearly documented examples.

### DependencyTrust checks

DependencyTrust should identify concerns such as:

- new dependency added without PR explanation
- dependency weakly connected to the diff
- dependency name looks typo-like or suspicious
- dependency overlaps with existing project capabilities
- dependency appears unused in changed code
- dependency introduces broad functionality for a narrow need
- dependency justification is missing or weak

V1 should avoid pretending to provide complete supply-chain security. The expected result is not a security verdict. The expected result is a clear dependency justification or a request for human review.

### DependencyTrust questions

DependencyTrust should ask targeted dependency-ownership questions.

Examples:

```text
This PR adds a new dependency. Explain why it is needed, where it is used, and why existing project functionality is not enough.
```

```text
This PR adds a dependency with broad functionality for a narrow change. Explain why the dependency is justified and what alternative was considered.
```

```text
This PR adds a dependency that appears unused in the changed code. Explain where it is used or remove it.
```

### DependencyTrust answer evaluation

A credible answer should usually include:

- the reason the dependency is needed
- where the dependency is used
- why existing dependencies or project functionality are insufficient
- basic trust or maintenance rationale
- evidence that the dependency is intentional

A weak answer usually:

- says only that the package is useful
- does not connect the dependency to changed code
- does not explain alternatives
- does not acknowledge risk
- cannot show where the dependency is used

## 10. Decision engine

The Decision Engine aggregates module outputs into a final status.

Suggested V1 decision rules:

```text
pass
- no blocking findings
- no required challenge remains unresolved
- all required answers are credible, if any were required

pass_with_warnings
- no blocking findings
- minor concerns remain
- answers are mostly credible

review_required
- at least one important answer is weak or incomplete
- human reviewer should inspect before merge

fail
- required answer is missing
- answer contradicts the change
- high-severity finding has no credible explanation
- dependency concern is unresolved and risky
```

Decision precedence in V1 should be simple and deterministic:

```text
fail > review_required > pass_with_warnings > pass
```

The final decision should be the most severe module decision unless a later version introduces explicit policy overrides.

The decision should be explainable. Reviewers should be able to see why ALFAID returned a result.

## 11. Output formats

V1 should support structured output first.

Recommended outputs:

- JSON for automation
- Markdown for reviewer readability
- terminal summary for local demo use

### JSON output shape

```json
{
  "decision": "review_required",
  "summary": "Retry behavior changed without sufficient duplicate-handling evidence.",
  "modules": [
    {
      "name": "ExplainGate",
      "findings": [
        {
          "severity": "high",
          "category": "retry_behavior",
          "summary": "Retry behavior changed in payment processing path."
        }
      ],
      "questions": [
        {
          "id": "Q1",
          "prompt": "Explain why the operation is safe to retry and which test proves duplicate execution is avoided."
        }
      ],
      "evaluations": [
        {
          "question_id": "Q1",
          "status": "weak",
          "rationale": "The answer mentions retries but does not explain idempotency or provide test evidence."
        }
      ]
    }
  ]
}
```

### Markdown output shape

```markdown
# ALFAID result: review_required

## Summary

Retry behavior changed without sufficient duplicate-handling evidence.

## Required reviewer attention

- ExplainGate: retry behavior changed in a sensitive path
- author answer did not explain idempotency
- no test evidence was provided

## Required next action

Author must explain duplicate-handling behavior and reference a test or fixture that proves it.
```

## 12. Demo scenarios

V1 should include deterministic scenarios that make the concept easy to review.

Recommended fixtures:

### Scenario 1: safe change

A small change with clear tests and no dependency changes.

Expected decision:

```text
pass
```

Expected behavior:

```text
No author challenge required.
```

### Scenario 2: risky behavior change with weak answer

A retry, validation, persistence, or error-handling change where the author gives a shallow answer.

Expected decision:

```text
review_required
```

Expected behavior:

```text
A small number of targeted questions are asked.
The weak answer fails because it lacks specific reasoning or evidence.
```

### Scenario 3: risky behavior change with strong answer

Same style of risky change, but with a credible explanation and evidence.

Expected decision:

```text
pass_with_warnings or pass
```

Expected behavior:

```text
The answer passes because it explains the actual behavior, assumption, relevant risk, and evidence.
```

### Scenario 4: new dependency without justification

A manifest adds a package, but the author does not justify why it is needed.

Expected decision:

```text
review_required or fail
```

### Scenario 5: suspicious or weakly motivated dependency

A dependency name is typo-like, unused, redundant, or too broad for the problem.

Expected decision:

```text
review_required
```

These scenarios should be easy to run and easy to understand from the output alone.

## 13. Fixture format

V1 should use deterministic scenario fixtures as the executable source of truth for tests and demos.

Each scenario should live under `fixtures/<scenario-name>/` and contain:

```text
fixtures/<scenario-name>/
├── pr.json
├── answers.json
└── expected.json
```

`pr.json` describes the PR-like input. `answers.json` contains author answers when the scenario includes them. `expected.json` defines the stable output fields that scenario tests should assert.

### pr.json

A minimal V1 `pr.json` should include:

```json
{
  "id": "PR-001",
  "title": "Change retry behavior for payment submission",
  "description": "Retries provider timeouts in the payment submission path.",
  "risk_profile": "high",
  "change_tags": ["retry", "idempotency", "payment"],
  "changed_files": [
    {
      "path": "src/payments/submitPayment.ts",
      "change_type": "modified",
      "summary": "Changed retry handling around provider timeout responses."
    }
  ],
  "dependency_changes": [],
  "test_evidence": [
    {
      "name": "PaymentRetryIdempotencyTest",
      "path": "tests/payment-retry-idempotency.test.ts",
      "summary": "Verifies duplicate provider submission is avoided when retrying after timeout."
    }
  ]
}
```

The exact field names may evolve with implementation, but fixtures should remain readable and stable enough for reviewers to understand without inspecting code internals.

### answers.json

A minimal V1 `answers.json` should include:

```json
{
  "answers": [
    {
      "question_id": "Q1",
      "answer_text": "The retry is safe because the operation uses an idempotency key stored before provider submission.",
      "evidence_refs": ["PaymentRetryIdempotencyTest"],
      "submitted_by": "author"
    }
  ]
}
```

A scenario may omit answers when testing the analysis-only phase or missing-answer behavior.

### expected.json

A minimal V1 `expected.json` should include only stable fields that are part of the behavior contract:

```json
{
  "decision": "review_required",
  "modules": {
    "ExplainGate": {
      "question_count": 1,
      "required_prompt_fragments": ["retry", "safe to retry", "duplicate execution"],
      "evaluation_statuses": ["weak"]
    }
  }
}
```

Scenario tests should avoid asserting incidental formatting, timestamps, or internal ordering unless ordering is part of the contract.

## 14. Design-driven TDD plan

Implementation should follow design-driven TDD in small vertical slices.

The pattern is:

```text
DESIGN.md defines the expected behavior
        ↓
A scenario fixture expresses one behavior
        ↓
A scenario test fails
        ↓
Implementation makes the scenario pass
        ↓
The next scenario is added
```

The goal is to keep implementation aligned with the product idea: ALFAID is an understanding gate, not a generic PR review bot.

### First vertical slice: ExplainGate

The first implementation slice should prove ExplainGate end to end using three fixtures:

```text
fixtures/safe-low-risk-change/
fixtures/retry-change-weak-answer/
fixtures/retry-change-strong-answer/
```

This slice should support:

- loading a scenario fixture
- building a `ChangeContext`
- running ExplainGate
- generating targeted questions
- loading author answers
- evaluating answers deterministically
- producing a final decision
- returning stable JSON output

#### safe-low-risk-change

Expected behavior:

- decision is `pass`
- ExplainGate emits no required questions
- no author answer is required
- output makes it clear that no challenge was needed

This scenario proves ALFAID does not challenge obvious low-risk changes.

#### retry-change-weak-answer

Expected behavior:

- decision is `review_required`
- ExplainGate emits exactly one required question
- the question references retry safety, duplicate execution, or evidence
- the author answer is evaluated as `weak`
- the rationale explains that idempotency or duplicate-handling evidence is missing

This scenario proves ALFAID does not accept shallow explanations for risky behavior changes.

#### retry-change-strong-answer

Expected behavior:

- decision is `pass` or `pass_with_warnings`
- ExplainGate emits the same retry-oriented question
- the author answer is evaluated as `credible`
- the answer references the relevant assumption and evidence
- the final output shows a clear path from explanation to decision

This scenario proves the gate is passable and fair when the author demonstrates understanding.

### Second vertical slice: DependencyTrust

After the ExplainGate slice works, V1 should add DependencyTrust using two fixtures:

```text
fixtures/dependency-added-no-justification/
fixtures/dependency-added-justified/
```

Expected behavior:

- a new dependency without justification leads to `review_required` or `fail`
- a justified dependency can pass when the answer explains why it is needed, where it is used, and why existing project functionality is insufficient
- the output clearly separates dependency ownership from full supply-chain security scanning

### Stable scenario assertions

Scenario tests should focus on stable product behavior:

- final decision
- module names
- finding categories
- question count
- required prompt fragments
- evaluation statuses
- rationale fragments for weak or missing answers
- recommended next actions

Scenario tests should not depend on incidental formatting, timestamps, generated IDs beyond the fixture contract, or exact Markdown rendering.

## 15. Implementation strategy

V1 should be built as a thin vertical slice, with tests written just ahead of each implementation step.

Recommended order:

1. define input fixture format
2. build ChangeContext parser
3. implement shared module contract
4. implement ExplainGate with deterministic triggers
5. implement challenge/question output
6. add author-answer input
7. implement deterministic answer evaluation
8. add Decision Engine
9. add DependencyTrust
10. add JSON and Markdown reports
11. add demo fixtures and tests
12. document reviewer path in README

The implementation should prioritize clarity over breadth.

## 16. Testing strategy

V1 should include tests for the assurance behavior, not only utility functions. The most important tests are scenario-level tests because they prove the product workflow end to end.

Recommended tests:

- ChangeContext parsing from fixtures
- ExplainGate emits expected questions for risky changes
- ExplainGate emits no challenge for safe low-risk changes
- ExplainGate asks no more than the configured maximum number of questions
- weak author answers are classified as weak or missing
- strong author answers are classified as credible
- answers are evaluated against the specific question, not generic writing quality
- DependencyTrust detects added dependencies
- DependencyTrust requires justification for new dependencies
- Decision Engine returns expected status for each fixture
- JSON output remains stable for deterministic scenarios


## 17. Trade-offs

### Targeted friction instead of universal enforcement

ALFAID should not challenge every change.

Reason:

- universal friction creates developer resistance
- obvious changes should remain fast
- targeted questions are more useful than generic questionnaires
- developers are more likely to accept a gate that clearly maps to risk

The risk is that V1 may miss some changes that deserve review. That is acceptable for the first version. The product should prefer being selective and explainable over noisy and annoying.

### Deterministic rules before advanced AI evaluation

V1 should prefer deterministic rules and fixtures over hidden model judgment.

Reason:

- easier to test
- easier to explain
- easier for reviewers to trust
- stronger portfolio signal for engineering discipline

LLM-based evaluation can be explored later, but V1 should not depend on opaque scoring to prove the core idea.

### Focused modules before broad coverage

V1 should implement two modules well instead of many modules shallowly.

Reason:

- ExplainGate proves the core understanding-gate concept
- DependencyTrust covers a distinct and realistic AI-assisted risk
- ArchitectureGuard can be stronger later once repository context handling exists

### Local fixtures before platform integration

V1 should use deterministic local fixtures before GitHub/GitLab integration.

Reason:

- easier to review
- easier to run in CI
- avoids spending early effort on integration plumbing
- keeps attention on the product idea

## 18. Non-goals

ALFAID V1 is not trying to:

- determine whether code was written by AI
- police whether an author used AI to draft an answer
- replace human reviewers
- fully understand arbitrary source code
- perform complete security analysis
- guarantee dependency safety
- enforce organization-wide governance
- block real production merges automatically
- solve every AI-assisted development risk

The narrow goal is stronger:

**Make risky changes require demonstrated understanding before merge.**

## 19. Success criteria

V1 is successful if a reviewer can run the demo and clearly see that:

- ALFAID detects review-sensitive changes
- ALFAID asks specific questions instead of generic comments
- safe low-risk changes are not burdened with unnecessary challenges
- weak answers fail to satisfy the gate
- strong answers can satisfy the gate
- dependency additions require justification
- final decisions are structured and explainable
- the project is not a generic AI code review assistant

The intended reviewer takeaway:

> ALFAID treats AI-assisted development risk as an assurance problem. It does not ask, "Did AI write this?" It asks, "Can the developer prove they understand this change well enough to merge it?"
