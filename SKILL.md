---
name: commander-mode
description: "Mandatory coordination for programming work and substantial read-only code analysis. Load after coding-discipline for code changes, or directly for debugging, review, and repository analysis that may benefit from delegation. It requires early complexity triage and normally delegates cross-cutting work with multiple independent risk domains to focused Pi workers using openai-codex/gpt-6-luna when Herdr is available. Skip pure Q&A, localized lookups, trivial edits, or when Herdr is unavailable."
---

# Commander Mode

The main agent is the commander and remains responsible for planning, integration, verification, and the final answer. Delegate bounded work only when it improves efficiency or coverage.

Apply the same commander policy when the main model is either `openai-codex/gpt-6-sol` or `github-copilot/claude-opus-5.5`. In both cases, keep architecture, integration, verification, and final approval with the main model, and delegate eligible worker tasks to Luna as specified below.

## Required Order and Preconditions

1. For code or test changes, load `coding-discipline` first. Skip it for read-only work as that skill directs.
2. Load this skill.
3. Before controlling workers, load `herdr` and follow its current CLI, geometry, focus, cwd, waiting, blocked-state, and safety instructions.

Project instructions and `coding-discipline` take precedence. Give workers all relevant project constraints.

Before any Herdr control command, verify:

```bash
test "${HERDR_ENV:-}" = 1
```

If unavailable, do not control Herdr; continue without delegation and mention this briefly. When syntax or agent support is uncertain, run `herdr --help` or `herdr agent`. Parse pane IDs from command JSON; never guess them.

## Complexity Gate

Classify the task before implementation:

- **Localized:** one coherent behavior, few dependencies, and end-to-end verification without crossing several architectural boundaries. Use no worker or one focused worker.
- **Cross-cutting:** coordinated changes across components or lifecycle stages; external contracts, stored state, compatibility, or operations are affected; or failure modes divide into multiple independent risk domains.

For cross-cutting work with at least two separable risk domains and available Herdr capacity, delegate during discovery and planning:

- Use one worker for a coherent deterministic trace, even across many files.
- Use two or three workers only for genuinely independent judgment-heavy domains. Avoid overlapping scans.
- Prefer read-only workers when edit ownership would overlap.
- Reserve architecture, security-sensitive decisions, shared invariants, integration, and final approval for the commander.
- When practical, use a separate post-change reviewer; this does not replace early domain decomposition.

Delegation may be skipped when separation would duplicate work, setup overhead outweighs the benefit, context cannot be shared safely, or the work must remain under commander control. State the reason.

Before proceeding, record:

```text
Complexity: localized | cross-cutting
Risk domains: <list>
Delegation: <worker scopes, or reason for skipping>
```

## Delegation Rules

Use workers for bounded discovery, call-path tracing, mechanical edits, explicit tests, focused checks, or independent review—not underspecified architecture, subtle security/concurrency decisions, conflict resolution, or final approval.

All workers must be Pi agents using provider `openai-codex`, model `gpt-6-luna`, and no skills:

```bash
herdr agent start <name> --kind pi --pane <pane-id> -- --provider openai-codex --model gpt-6-luna --no-skills
```

Use the lowest safe reasoning effort:

- `low`: lookup, command execution, summaries, mechanical edits.
- `medium`: bounded implementation, straightforward tests, review.
- `high`: subtle bounded logic or revision after evidence of a reasoning gap.

Escalate only after inspecting evidence; prefer a focused follow-up over restarting a worker.

Create workers only in panes assigned to this task. Give concurrent editing workers disjoint files; otherwise serialize them or make one read-only. Never let two workers edit the same file concurrently.

Every worker prompt must specify:

- Exact goal, acceptance criteria, owned files or subsystem, and whether editing is allowed.
- Constraints, conventions, and required checks.
- `Do not load or invoke optional skills.`
- No unrelated cleanup, commits, pushes, rebases, or destructive Git commands.
- A concise final report with changed files, evidence locations, checks and results, assumptions, and unresolved risks—not narrative progress logs.

Do not ask workers to read a `SKILL.md` unless the user explicitly requires it. Do not override automatically applied mandatory instructions.

## Supervision and Review

Track workers by unique names. Use `herdr agent prompt ... --wait` and inspect settled state. If blocked, inspect terminal output; never approve security or permission prompts without user authorization. Redirect scope drift promptly.

A precise, cited worker report is sufficient for deterministic read-only discovery; request a focused follow-up for gaps or contradictions instead of repeating the scan. Independently verify ambiguous runtime, architecture, or security conclusions.

For every code change, the commander must:

1. Inspect `git status`, the complete diff, and all changed code in context.
2. Attribute every changed line to the request and preserve unrelated user changes.
3. Check correctness, simplicity, conventions, edge cases, security, and test quality.
4. Run relevant checks independently; worker-reported results alone are insufficient.
5. Resolve valid findings and repeat review and verification as needed.

A review worker supplements but never replaces the commander's review.

## Completion

Treat each Herdr pane created for a worker as scoped to that worker's assigned task. As soon as its work is finished and its output has been collected, or the pane is otherwise no longer needed, stop the worker and close the pane immediately; do not leave unused panes open until the overall programming task ends. Before the final response, check that no panes created by the commander for this task remain open. Never close panes that existed before the task began or were not created by the commander.

The final response should briefly state:

- What changed.
- What was delegated, if anything.
- What the commander reviewed or corrected.
- Checks run and their outcomes.
- Any unresolved risk or limitation.
