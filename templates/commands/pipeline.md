---
description: Run the full autonomous speckit pipeline — specify through implement with quality gates.
scripts:
  sh: scripts/bash/check-prerequisites.sh --json --paths-only
  ps: scripts/powershell/check-prerequisites.ps1 -Json -PathsOnly
---

## User Input

```text
$ARGUMENTS
```

The user input is the **feature description** (required). Optional flags:
- `--commit` — after implementation, create a git commit, push, and open a PR

Parse the arguments: extract the feature description text and whether `--commit` is present.

## Goal

Run the complete speckit pipeline autonomously — from specification through implementation — with convention-based quality gates. No human checkpoints. Halts only on unresolvable blockers (writes `blockers.md`).

## Execution Steps

### 1. Initialize

Run `{SCRIPT}` once from repo root to get `REPO_ROOT`.

Extract the feature description and `--commit` flag from user input.

### 2. Specify

Run `/speckit.specify {feature-description}`.

This creates the feature branch, spec directory, and `spec.md`.

### 3. Resolve Feature Directory

Re-run `{SCRIPT}` (without `--require-tasks`) to get `FEATURE_DIR` now that the feature branch and spec exist.

### 4. Convention Detection

Check which autonomous infrastructure is available:

```
AUTONOMOUS_CONSTITUTION = exists(".specify/memory/autonomous-constitution.md")
   — OR constitution.md contains "## Autonomous Clarification Protocol"
QUALITY_GATE = exists(".specify/memory/quality-gate.md")
DECISIONS_TEMPLATE = exists(".specify/templates/decisions-template.md")
PLAYWRIGHT_CONFIG = exists("playwright.config.js") OR exists("playwright.config.ts")
E2E_SKILL = /speckit.e2e is available as a skill
```

Each gate is independently enabled. Missing infrastructure means that gate is skipped (not an error).

### 5. Self-Clarification Loop (if AUTONOMOUS_CONSTITUTION)

Read `autonomous-constitution.md` (or the autonomous sections in `constitution.md`) for the clarification protocol.

1. Re-read the generated `spec.md` in full.
2. Adopt the role of a skeptical Product Manager — identify ambiguous, missing, or contradictory requirements.
3. For each issue, resolve using the most conservative interpretation.
4. Write all resolutions to `{FEATURE_DIR}/decisions.md` (use `decisions-template.md` if available).
5. Do NOT ask for human input. Resolve autonomously.

### 6. Quality Gate Loop (if QUALITY_GATE)

Read `.specify/memory/quality-gate.md`.

1. Score `spec.md` against every checklist item — mark PASS or FAIL with a one-line rationale.
2. For each FAIL: fix `spec.md` immediately.
3. Re-score until all items PASS.
4. Write final scores to `{FEATURE_DIR}/quality-report.md`.

If after 3 full cycles any item still fails: write `{FEATURE_DIR}/blockers.md` and **halt**.

### 7. Plan

Run `/speckit.plan`.

### 8. Tasks

Run `/speckit.tasks`.

### 9. Task Structural Validation (if AUTONOMOUS_CONSTITUTION)

Read the Task Structural Validation section from `autonomous-constitution.md` (or `constitution.md`).

Validate `tasks.md` against:

**Coverage:**
- Every user story in `spec.md` maps to at least one task.
- Every acceptance criterion is addressed by at least one task.

**Structure:**
- Every task specifies at least one target file path.
- Every task has a clear success condition.
- No open questions or unresolved references.

**Ordering:**
- No dependency on a later task.
- Parallel tasks `[P]` don't share write targets.

**Autonomy:**
- Every task can be implemented without human input.

If validation fails: fix `tasks.md` and re-validate. If still failing after 3 cycles: write `{FEATURE_DIR}/blockers.md` and **halt**.

### 10. Pre-Flight Assertions (if AUTONOMOUS_CONSTITUTION)

Read the Pre-Flight Assertions section. Assert ALL of:

- [ ] `constitution.md` exists
- [ ] `spec.md` exists and has no unchecked items
- [ ] `decisions.md` exists (if self-clarification was enabled)
- [ ] `quality-report.md` exists and all PASS (if quality gate was enabled)
- [ ] `tasks.md` passed structural validation (if task validation was enabled)
- [ ] No `blockers.md` with unresolved items

If any fail: write `{FEATURE_DIR}/blockers.md` and **halt**.

### 11. Implement

Run `/speckit.implement`.

### 12. E2E Tests (conditional)

**Only if** ALL of these are true:
- PLAYWRIGHT_CONFIG exists
- E2E_SKILL is available
- `plan.md` references frontend files (e.g., files under `client/src/`, `src/pages/`, `src/components/`)

Then run `/speckit.e2e`.

If tests fail after 3 retry cycles: write `{FEATURE_DIR}/blockers.md` and **halt**.

**Skip** if any condition is false (not an error — just skip silently).

### 13. Commit and PR (if --commit flag)

1. Use the `/commit` skill to create the git commit. Do NOT create commits manually.
2. Push the branch: `git push -u origin HEAD`
3. Create PR: `gh pr create --fill`

If `/commit` skill is not available, fall back to manual `git add` + `git commit`.

### 14. Report

Display a summary of what was done:

```
## Pipeline Complete: {feature-description}

### Steps Executed
- [x] Specify — spec.md created
- [x] Self-clarification — decisions.md ({N} decisions)  [or: skipped — no autonomous constitution]
- [x] Quality gate — all items PASS  [or: skipped — no quality-gate.md]
- [x] Plan — plan.md created
- [x] Tasks — tasks.md created ({N} tasks)
- [x] Task validation — all rules pass  [or: skipped]
- [x] Pre-flight — all assertions pass  [or: skipped]
- [x] Implement — all tasks executed
- [x] E2E — tests pass  [or: skipped — no playwright config / backend-only]
- [x] Commit + PR — {PR_URL}  [or: skipped — no --commit flag]

### Artifacts
- Spec: {FEATURE_DIR}/spec.md
- Decisions: {FEATURE_DIR}/decisions.md
- Quality report: {FEATURE_DIR}/quality-report.md
- Plan: {FEATURE_DIR}/plan.md
- Tasks: {FEATURE_DIR}/tasks.md
```

## Halting Protocol

At no point during steps 1-13 should the agent pause for human input.

If an unresolvable blocker is hit at any step:
1. Write `{FEATURE_DIR}/blockers.md` with the specific failure and context.
2. Display the blocker to the user.
3. **Stop.** Do not continue to the next step.

Never ask for human clarification mid-pipeline. Resolve autonomously or halt.

## Operating Principles

- **Convention over configuration** — the pipeline auto-detects available infrastructure. No flags needed to enable gates.
- **Graceful degradation** — missing infrastructure means the gate is skipped, not failed. A project with zero autonomous files still runs: specify → plan → tasks → implement.
- **Fail fast, fail loud** — blockers halt immediately with a clear report. No silent failures.
- **Idempotent gates** — each validation loop has a max retry count (3). Infinite loops are impossible.
