---
name: superthink
description: Clarify requirements, understand the problem, and plan implementation when ambiguity or consequential trade-offs risk building the wrong thing; investigate bugs, troubleshoot exceptions, analyze failures and regressions through root cause analysis; review code, validate implementation, check missing cases, and verify correctness against requirements. Use for these reasoning needs, including before delivering an implementation or fix, not routine explanations, mechanical edits, or clear low-risk tasks.
---

# superthink

Apply the least reasoning process needed to reach a correct, evidenced outcome. This is a standing reasoning discipline, not a fixed workflow or three user-facing commands. Use Claude Code's existing search, reading, editing, execution, diff, and testing capabilities; require no other skill, reference, script, CLI package, MCP, or plugin. Use project tools when available; do not invent replacements for native planning, todos, git, tests, or subagents.

## Shared rules

- Understand before changing. Ask the codebase before asking the user.
- Evidence before conclusions. Distinguish observed facts, hypotheses, and assumptions.
- Root cause before fixes. Requirements before review. Verification before confidence.
- Ask only when necessary. Stop when sufficient. Preserve the user's scope and existing authorization.
- Share concise conclusions and supporting evidence, not an internal reasoning transcript or a ceremonial checklist. Match the user's language.

## Quiet routing and depth

Choose the immediate reasoning need from context; reassess when new evidence changes it. Do not ask the user to choose a mode or routinely announce mode names.

| Current need | Internal route |
| --- | --- |
| Desired new behavior with uncertainty; feature, API, module, architecture, refactor, or planning with meaningful trade-offs or broad impact | UNDERSTAND |
| Unexpected current behavior; bug, exception, crash, failing test/CI, regression, sudden slowdown, inconsistent data, race, or investigation request | DEBUG |
| Assess existing work; review, PR/diff, missing cases, “is this correct?”, or implementation/fix ready for delivery | VERIFY |
| Explanation, summary, or clear mechanical task without these risks | Normal assistance; no protocol |

For overlapping requests, start with the immediate blocker: bug + fix → DEBUG → fix → VERIFY; new feature + implementation → UNDERSTAND → implement → VERIFY; existing diff + review → VERIFY. A failure during implementation or verification leads to DEBUG; a contract ambiguity that changes the solution calls for focused UNDERSTAND. Resume the authorized task afterwards. A plan-only request ends with a plan; a review-only request reports findings without silently editing code.

Scale depth to uncertainty, impact, and reversibility:

- **Level 1:** Clear, local, low-risk work: act directly and make a proportionate check. No interview or formal plan.
- **Level 2:** Some uncertainty: inspect relevant code and contracts, resolve the material gap, use targeted evidence and verification.
- **Level 3:** Complex, high-risk, or cross-module work: trace boundaries, clarify consequential decisions, and use the fuller applicable guidance below. Never run all modes just because the task is large.

## UNDERSTAND — what problem are we solving?

Read relevant code, tests, docs, callers, architecture, and conventions first. Infer test framework, file organization, API patterns, dependency injection, and error handling from the repository. Inspect only what can affect the decision.

Establish enough of the following to act:

- **Problem / outcome:** Why change anything, and what observable behavior should change? Distinguish a requested solution from its underlying problem without overriding an explicit user choice.
- **Success criteria:** Concrete accepted behavior, including the important failure path.
- **Constraints / scope:** What must hold, what is excluded, and where the implementation ends.
- **Trade-offs / existing contract:** Which of correctness, speed, compatibility, complexity, and performance matters here? Which existing behaviors must remain?

Look for material hidden assumptions: compatibility, consistency, failure strategy, migration, performance bounds, and security boundaries. Treat these as investigation prompts, not extra requirements to invent.

Ask only if the answer cannot be obtained from the repository/context, multiple reasonable answers remain, and choosing wrongly would materially change the approach or cause rework. Ask the single most consequential question first. Use `Question Cost < Wrong-Assumption Cost`; do not turn this into a questionnaire or ask about already settled decisions.

If told “don't ask, just do it,” proceed with reasonable assumptions and briefly label material ones `Assumption:`. For an unresolved destructive operation, irreversible migration, major public API change, security-sensitive assumption, or completely ambiguous goal, state the concrete uncertainty; pause only the dependent action when missing intent or authorization makes proceeding unsafe. Do independent work and do not ask again for authorization already given.

**Exit:** Problem, outcome, important constraints, acceptance criteria, relevant architecture, and implementation boundary are sufficiently clear. Stop asking when remaining uncertainty will not change the approach; do not chase a confidence percentage.

For substantial work, give a concise executable plan covering: Goal; Relevant Existing Behavior; Proposed Change; Files / Components likely affected; Important Constraints; Implementation Steps; Verification; Risks / Assumptions. Name concrete components, behavior changes, and checks instead of “change backend, add tests.” Compress or omit the template for simple work. If implementation is authorized, continue without a ceremonial approval gate.

## DEBUG — what causes the failure?

**Do not fix what you do not understand.** Before changing production logic, establish an evidence-supported causal explanation. A plausible patch or a green test alone is not a root cause.

1. Compare observed and expected behavior. Read the full error, stack trace, failing assertion, inputs, environment, and relevant contract. Record a reproduction or the conditions under which failure occurs.
2. Locate the failure boundary. Trace call/data flow from the symptom toward the first invalid state or broken contract. Inspect relevant logs, runtime state, recent diff/history, and working versus failing cases. Use boundary instrumentation, a minimal reproduction, or bisect when it narrows the search; do not collect everything indiscriminately.
3. Form one primary falsifiable hypothesis tied to evidence. Specify the predicted observation and smallest experiment that distinguishes it from alternatives. Run it, record the result, and retain or reject the hypothesis before trying the next. Do not bundle unrelated speculative edits.
4. Confirm the causal chain: trigger → mechanism → violated contract → symptom. For intermittent/concurrent failures, examine event order and use controlled scheduling or repeated reproduction where practical; one passing rerun does not establish a fix.
5. When test infrastructure permits, add a meaningful regression test and observe it fail for the original reason before the fix. Then make the smallest local correction at the responsible contract boundary, following the existing design. Verify the regression passes and switch to VERIFY for adjacent risks.

Temporary diagnostic changes or isolated experiments are allowed before confirmation; they are not a fix. Remove obsolete instrumentation and speculative changes you introduced without discarding user work. Do not hide symptoms with blanket null guards, swallowed exceptions, arbitrary retries, changed assertions, unrelated refactors, new abstractions, or unrequested API changes.

For example, a null error requires tracing where null first appeared and whether the contract permits it. A guard is justified only if null is valid and that boundary owns its handling.

If reproduction is unavailable, use the strongest available trace/code evidence and state its limits. Label an unconfirmed cause as a hypothesis. If experiments stop producing new evidence, reassess the boundary and assumptions; request the smallest missing artifact or report the blocker instead of cycling through patches. Distinguish temporary mitigation from a root-cause fix.

**Exit:** The root cause is supported by evidence and the fix passes relevant verification. If blocked, stop with known facts, remaining uncertainty, and the next discriminating check; do not claim resolution. Do not generate low-value tests solely to satisfy the sequence. Explain when a pre-fix failure or regression test could not be obtained.

## VERIFY — does the implementation meet the need?

Re-read the original request, subsequent accepted clarifications, acceptance criteria, relevant design decisions, current changes, and actual test results. Establish the review scope/base from context or repository evidence; include relevant staged, unstaged, committed, and new files as applicable. Read surrounding code and callers, not only changed lines. An empty working-tree diff does not prove there is nothing to review. If requirements are unavailable, state the inferred contract and limit claims of completeness.

Map each important requirement to **implementation → evidence**. Identify gaps where code exists but behavior is not demonstrated. Consider all dimensions below for relevance; inspect the ones the change can affect:

| Dimension | Questions that matter |
| --- | --- |
| Correctness | Does behavior satisfy the requirement? Are happy path, state transitions, and data flow correct? |
| Missing cases | Relevant empty/null/zero/duplicate inputs; partial failure, timeout, retry, cancellation, concurrency, race, ordering, stale state? |
| Error handling | Correct propagation and fallback? Swallowed errors or leaked internal information? |
| Compatibility | Existing behavior, public API, data formats, and migration preserved or intentionally changed? |
| Architecture | Existing abstractions, layer ownership, dependency direction respected? Duplicated logic or unnecessary complexity? |
| Security, when relevant | Authentication, authorization, validation, injection, secrets, path traversal, unsafe deserialization, information leakage? |
| Performance, when relevant | N+1, unbounded work, repeated computation, memory growth, unnecessary network calls, blocking? |
| Tests | Assertions prove key behavior, including failure paths? Mocks hide the logic under test? Regression coverage addresses the original bug? |

Run cost-appropriate project checks when available: targeted tests first, then affected integration tests, typecheck, lint, or build as warranted. Static checks do not prove runtime correctness. Broaden checking for unresolved risk; do not run every command ritualistically. After a correction, rerun affected checks against the final code. Never report a command as passed if it was not run or failed; distinguish environment failures from implementation failures.

Report actionable findings with severity, location when available, triggering condition, impact, and evidence. Separate speculative risks from demonstrated defects; order by impact:

- **CRITICAL:** Severe wrong results, security compromise, data corruption, or severe regression.
- **IMPORTANT:** Real bug, meaningful requirement omission, or concrete architecture risk.
- **MINOR:** Local maintainability or quality issue.
- **OPTIONAL:** Style or preference; usually omit unless requested.

Prioritize CRITICAL / IMPORTANT; do not manufacture nitpicks. If there are no substantive findings, say “没有发现影响正确性的明显问题” or its equivalent in the user's language, bounded by review scope and evidence.

**Exit:** Requirements, key paths, relevant risks, and meaningful tests have been checked, or unavailable checks are explicitly identified. End with a concise result/findings plus **Verified** (checks and outcomes) and **Not verified** (gaps and reasons, or none within scope). Do not imply exhaustive correctness. Stop unless new evidence justifies further investigation.
