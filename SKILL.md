---
name: commander-mode
description: "Mandatory, credit-aware coordination mode for programming work and substantial read-only code analysis. Load after coding-discipline for code changes, and load directly for debugging, code review, or repository analysis large enough to benefit from delegation. It requires explicit complexity triage and early decomposition; delegation is normally required for cross-cutting work with multiple independent risk domains when Herdr is available. The main agent delegates bounded discovery and review exclusively to lower-cost Pi workers using openai-codex/gpt-5.6-luna while retaining final accountability. Skip pure Q&A, small localized code lookups, trivial typo/config edits, or when Herdr is unavailable."
---

# Commander Mode

The main agent is the commander and final owner of the result. Loading this mode is mandatory when it applies. Delegation follows the complexity gate below: it may be skipped for localized work, but is normally required for cross-cutting work with multiple separable risk domains when Herdr is available. Workers provide implementation or analysis capacity; they do not replace the commander's judgment.

## Required Skill Order

When this skill applies:

1. For code or test changes, load `coding-discipline` first and obey it throughout the task. Skip it for read-only analysis, as that skill directs.
2. Load this skill.
3. Before controlling workers, load the `herdr` skill and follow its current CLI and safety instructions exactly.

If instructions conflict, project instructions and `coding-discipline` take precedence over this skill. Tell every worker the relevant project constraints.

## Preconditions

Before any Herdr control command, verify:

```bash
test "${HERDR_ENV:-}" = 1
```

If it fails, do not control Herdr from outside Herdr. Continue the task yourself and state briefly that delegation was unavailable.

Use `herdr --help` and `herdr agent` when syntax or supported agent kinds may have changed. Never assume pane IDs; parse them from command JSON.

## Cost Model

Optimize for total credits and result quality, not raw token count. Worker tokens can be economical when the worker model is materially cheaper than the commander.

Current reference pricing per 1M tokens:

| Token type | Luna | Sol |
|---|---:|---:|
| Input | 20 | 200 |
| Output | 120 | 1000 |
| Cache read | 2 | 20 |
| Cache write | 25 | 250 |

At these rates, Luna input and cache traffic cost one tenth of Sol, and Luna output costs about one eighth. A Luna worker may therefore consume several times more tokens and still reduce total credits. Re-evaluate this assumption if the active commander model or provider pricing changes; inspect `PI_MODEL` when relevant.

Use this asymmetry deliberately:

- Offload broad searches, inventory, call-path tracing, test-gap discovery, and first-pass review to Luna.
- Ask workers for concise conclusions with exact files, symbols, line references, commands, and evidence.
- For deterministic read-only discovery such as symbol lookup and call-stack tracing, treat a worker's cited report as the discovery result. Do not reopen the cited source merely to repeat the same trace.
- Use one Luna worker for a coherent read-only trace even when it crosses many files. Add workers only for genuinely independent judgment-heavy risk domains, not duplicate discovery.
- Keep architecture decisions, disputed findings, edits, complete-diff ownership, and final approval with the commander.
- Avoid overlapping broad worker scopes: cheap duplication is still waste when it produces no independent coverage.

A cited worker report is sufficient for mechanical read-only discovery. Ask the same worker a focused follow-up when evidence is missing or internally inconsistent. Independent commander verification remains required for code changes and for conclusions that depend on ambiguous runtime behavior, security judgment, or architecture rather than a deterministic source trace.

## Commander's Workflow

### 1. Understand and plan first

Before delegating:

- Read project guidance and the minimum entry-point code needed to understand the requested outcome; delegate broad discovery when the complexity gate supports it.
- State assumptions or ask about genuine ambiguity.
- Define concrete success criteria and relevant verification.
- Split work into bounded tasks with explicit ownership.
- Keep architecture, risky decisions, security-sensitive logic, cross-cutting invariants, and final integration under commander control.

Do not delegate merely to appear busy. For a tiny change, direct implementation is cheaper and safer.

### 2. Classify complexity and decide delegation

Classify the task before implementation, not only when it is time for final review.

Treat work as **localized** when it is confined to one coherent unit of behavior, has few dependencies, and can be understood and verified end to end without tracing several architectural boundaries. One worker or no worker is normally appropriate.

Treat work as **cross-cutting** when any of these apply:

- It changes an invariant across multiple architectural boundaries.
- Correctness depends on coordinated behavior in several components or lifecycle stages.
- It changes externally visible contracts, stored state, compatibility expectations, or operational procedures.
- It has multiple independent verification dimensions or a broad change surface.
- Its failure modes naturally divide into two or more distinct risk domains.

For cross-cutting work with at least two separable risk domains and available Herdr capacity:

1. Delegate early, during discovery and planning—not only after implementation.
2. Use one worker when the task is a coherent deterministic read-only trace, even if it crosses many files.
3. Use two or more focused workers only when scopes require genuinely independent judgment. Prefer two or three; exceed three only when the decomposition clearly warrants it.
4. Give each additional worker one risk domain, such as invariant analysis, state-transition review, compatibility analysis, operational impact, or test-gap analysis. Do not split mechanical dependency tracing into overlapping workers.
5. Prefer read-only discovery workers when file ownership would overlap; the commander should use their cited evidence without duplicating their source scan.
6. For code changes, reserve a separate post-implementation reviewer when practical, but keep that review focused on the acceptance criteria and changed diff. A single broad final reviewer does **not** substitute for early domain decomposition when multiple judgment-heavy risk domains exist.

Delegation may still be skipped when tasks cannot be separated without duplication, worker setup would cost more than the task, relevant context cannot safely be shared, or all work is security-sensitive architecture that must remain with the commander. Record that reason in the working plan instead of silently defaulting to one or zero workers.

Before proceeding, the commander should be able to state:

```text
Complexity: localized | cross-cutting
Risk domains: <list>
Delegation: <workers and scopes, or explicit reason for skipping>
```

### 3. Delegate bounded work

Good worker tasks include:

- Locating symbols and tracing a narrow call path.
- Adding straightforward tests from explicit acceptance criteria.
- Implementing a small, well-specified change in isolated files.
- Mechanical migrations or repetitive edits.
- Running focused checks and summarizing failures.
- Performing an independent read-only review of a diff.

Do not delegate underspecified architecture, subtle concurrency/security work, final conflict resolution, or the final approval decision.

Choose the lowest reasoning effort that safely fits the delegated task:

- `low`: symbol searches, focused command execution, failure summaries, and purely mechanical edits.
- `medium`: default for bounded implementation, straightforward tests, and independent code review.
- `high`: subtle but still bounded local logic, or a revision after a medium-effort attempt exposed a real reasoning gap. Work requiring greater effort belongs with the commander.

All delegated workers must use Pi with provider `openai-codex` and model `gpt-5.6-luna`. Do not start Codex or other worker models. Start Pi without loading any skills:

```bash
herdr agent start <name> --kind pi --pane <pane-id> -- --provider openai-codex --model gpt-5.6-luna --no-skills
```

Do not increase effort preemptively. Escalate only after inspecting evidence that the current level is insufficient; prefer a precise follow-up prompt over restarting a worker at higher effort.

Start workers only in panes created or explicitly assigned for this task. Follow the Herdr skill's geometry, `--no-focus`, cwd, waiting, blocked-state, and safety rules.

Use the smallest count that covers the identified risk domains—not merely the smallest possible count. Prefer one worker for a narrow task. For cross-cutting work, prefer two or three workers with non-overlapping scopes, often followed by a separate reviewer. Do not create multiple broad workers that all scan the same diff.

#### Worker skill isolation

Workers must not proactively load, invoke, or follow optional skills. Pi workers must be started with `--no-skills`. In particular, do not have workers load orchestration or repository-analysis skills such as `commander-mode`, `herdr`, or `project-code-memory`; doing so duplicates context, increases cost, and can recursively expand the task. Workers should follow only their non-overridable system/developer instructions, mandatory project instructions, and the commander's bounded task prompt.

Every worker prompt must explicitly say: `Do not load or invoke optional skills.` Never ask a worker to read a `SKILL.md` unless the user specifically requires that skill for the delegated work. If a worker runtime automatically applies a mandatory skill or instruction, do not attempt to override it; prohibit only additional optional skill loading.

When starting a Pi worker, always pass `--no-skills`; only omit it if the user explicitly requests worker skills. Retain the prompt-level prohibition as well.

### 4. Give precise orders

Every worker prompt must include:

- Exact goal and acceptance criteria.
- Files or subsystem it owns.
- Relevant constraints and existing conventions.
- Whether it may edit or must remain read-only.
- Required tests/checks.
- A prohibition on loading or invoking optional skills.
- A prohibition on unrelated cleanup, commits, pushes, rebases, and destructive Git commands.
- A request for a concise report containing changed files, exact evidence locations, checks run, results, and remaining concerns; discourage narrative progress logs and unrelated observations.

Example:

```text
Implement only <bounded change> in <owned files>.
Acceptance criteria: <observable behavior>.
Follow <project constraints>. Do not load or invoke optional skills. Do not touch
unrelated files, commit, push, rebase, or run destructive Git commands. Run
<focused checks>. When done, report changed files, test results, assumptions,
and any unresolved risk.
```

For shared working trees, assign disjoint files whenever multiple workers may edit. Never let two workers concurrently edit the same file. If ownership cannot be isolated, serialize the work or make one worker read-only.

### 5. Supervise actively

- Track each worker by its unique agent name.
- Use `herdr agent prompt ... --wait` and inspect settled state.
- If a worker is blocked, read its terminal output. Do not answer approval or security prompts without user authorization.
- Redirect a worker when it exceeds scope, misunderstands requirements, or performs unrelated cleanup.
- For deterministic read-only lookup or call-stack tracing, accept a concise report with exact citations as evidence and do not repeat the source scan. Use a focused follow-up only when the report exposes a gap, conflict, or uncertainty.
- For worker edits, inspect the actual working tree and complete diff; a summary never substitutes for code review.

Workers may be asked to revise their own bounded work. The commander must independently validate worker-produced code and judgment-heavy conclusions, but not mechanically retrace a complete read-only call stack.

### 6. Review and integrate

After a worker edits code, the commander must:

1. Inspect `git status` and the complete diff.
2. Attribute every changed line to the requested goal and detect overlapping/unexpected edits.
3. Read all changed code in context.
4. Check correctness, simplicity, conventions, edge cases, security, and test quality under `coding-discipline`.
5. Run relevant tests/checks independently; worker-reported results are not sufficient.
6. Fix issues directly or send a precise revision request, then repeat review and verification.
7. Remove only artifacts introduced by this task; preserve unrelated user changes.

For nontrivial changes, use a separate Pi `openai-codex/gpt-5.6-luna` worker as an independent read-only reviewer when practical. This review complements—not replaces—the early risk-domain workers required by the complexity gate. Give the reviewer the acceptance criteria, the commander's intended invariants, known assumptions, and the changed-file scope. Ask for concise actionable findings with exact evidence, not another unrestricted repository analysis. The commander decides which findings are valid and ensures they are resolved.

## Final Accountability

The commander—not Pi workers or Herdr—is responsible for the final answer and code quality. Code-change success must never be claimed solely from worker output. Deterministic read-only discovery may rely directly on a worker's precise, cited report without duplicate commander verification.

The final response should briefly report:

- What was changed.
- What work was delegated, if any.
- What the commander reviewed or corrected.
- Tests/checks run and their outcomes.
- Any unresolved risk or limitation.
