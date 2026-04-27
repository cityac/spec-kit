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
- `--scope XS|S|M|L` — override scope classification (default: M). Controls which stages run:

| Scope | Stages | Use Case |
|-------|--------|----------|
| **XS** | implement → commit | Typo fix, config tweak, one-file change. No spec needed. |
| **S** | specify → implement → commit | Small feature, clear scope. Skip plan/tasks/clarification/quality gate. |
| **M** | Full pipeline (default) | Standard feature. All stages run. |
| **L** | Full pipeline + cost/risk gate | Large feature. All stages + explicit effort review before implement. |

Parse the arguments: extract the feature description, `--commit` flag, and `--scope` value (default M if not provided).

## Goal

Run the complete speckit pipeline autonomously — from specification through implementation — with convention-based quality gates. No human checkpoints. Halts only on unresolvable blockers (writes `blockers.md`).

The pipeline adapts its depth based on `--scope`:
- **XS**: Jump straight to Step 11 (implement). No spec, plan, or tasks.
- **S**: Run Steps 2, 3, then jump to Step 11 (implement). Skip clarification, quality gate, plan, tasks.
- **M**: Run all steps (default, current behavior).
- **L**: Run all steps + Step 10.5 (cost/risk gate before implement).

## Execution Steps

### 1. Initialize

Run `{SCRIPT}` once from repo root to get `REPO_ROOT`.

Extract the feature description and `--commit` flag from user input.

### 1.5. Scope Routing

Based on the `--scope` value, determine which steps to execute:

- **XS**: Skip to Step 11 (Implement). The feature description IS the implementation instruction. No feature branch, no spec directory — just edit, test, done.
- **S**: Run Step 2 (Specify) → Step 3 (Resolve Feature Directory) → Step 11 (Implement) → Step 12 (E2E) → Step 13 (Commit). Skip Steps 4-10.
- **M**: Run all steps 2-13 as written below (default).
- **L**: Run all steps 2-13 + Step 10.5 (Cost/Risk Gate).

For **XS scope**: create a minimal feature branch (`git checkout -b fix/{slugified-description}`) and skip directly to Step 11. After implementation, run any fast validation (lint, type check) and proceed to Step 13 if `--commit`.

For all other scopes, continue to Step 2.

### 2. Specify (skip if XS)

Run `/speckit.specify {feature-description}`.

This creates the feature branch, spec directory, and `spec.md`.

### 3. Resolve Feature Directory (skip if XS)

Re-run `{SCRIPT}` (without `--require-tasks`) to get `FEATURE_DIR` now that the feature branch and spec exist.

### 4. Convention Detection (skip if XS or S)

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

### 5. Self-Clarification Loop (if AUTONOMOUS_CONSTITUTION; skip if XS or S)

Read `autonomous-constitution.md` (or the autonomous sections in `constitution.md`) for the clarification protocol.

1. Re-read the generated `spec.md` in full.
2. Adopt the role of a skeptical Product Manager — identify ambiguous, missing, or contradictory requirements.
3. For each issue, resolve using the most conservative interpretation.
4. Write all resolutions to `{FEATURE_DIR}/decisions.md` (use `decisions-template.md` if available).
5. Do NOT ask for human input. Resolve autonomously.

### 6. Quality Gate Loop (if QUALITY_GATE; skip if XS or S)

Read `.specify/memory/quality-gate.md`.

1. Score `spec.md` against every checklist item — mark PASS or FAIL with a one-line rationale.
2. For each FAIL: fix `spec.md` immediately.
3. Re-score until all items PASS.
4. Write final scores to `{FEATURE_DIR}/quality-report.md`.

If after 3 full cycles any item still fails: write `{FEATURE_DIR}/blockers.md` and **halt**.

### 7. Plan (skip if XS or S)

Run `/speckit.plan`.

### 8. Tasks (skip if XS or S)

Run `/speckit.tasks`.

### 9. Task Structural Validation (skip if XS or S)

Structural validation now runs inside `/speckit.tasks` (Step 5 of tasks.md). Check that `{FEATURE_DIR}/task-validation-report.md` exists and all rules passed.

If the report shows failures: write `{FEATURE_DIR}/blockers.md` and **halt**.

### 10. Pre-Flight Assertions (if AUTONOMOUS_CONSTITUTION; skip if XS or S)

Read the Pre-Flight Assertions section. Assert ALL of:

- [ ] `constitution.md` exists
- [ ] `spec.md` exists and has no unchecked items
- [ ] `decisions.md` exists (if self-clarification was enabled)
- [ ] `quality-report.md` exists and all PASS (if quality gate was enabled)
- [ ] `tasks.md` passed structural validation (if task validation was enabled)
- [ ] No `blockers.md` with unresolved items

If any fail: write `{FEATURE_DIR}/blockers.md` and **halt**.

### 10.5. Cost / Risk Gate (L scope only)

**Only runs when `--scope L`** or when auto-detected thresholds are exceeded.

Compute from `tasks.md`:
- **Total task count**
- **Distinct files touched** (union of all target file paths)
- **Parallel-able count** (tasks marked `[P]`)
- **Sensitive paths**: any task targeting files matching these patterns:
  - `**/migrations/**`, `**/migrate*`
  - `**/auth/**`, `**/middleware/auth*`
  - `**/payment*`, `**/billing*`, `**/stripe*`
  - `**/deploy*`, `**/.github/**`, `**/ci*`, `**/.gitlab*`
  - `**/.env*`, `**/secrets*`

**Hard gate** — halt with `{FEATURE_DIR}/effort-report.md` if ANY of:
- `total_tasks > 25` AND scope was not explicitly set to L (i.e., was auto-classified or defaulted to M)
- Any sensitive path matched AND scope was not explicitly set to L

`effort-report.md` contains:
```
## Effort & Risk Report
- Total tasks: N
- Files touched: N
- Sensitive paths: [list or "none"]
- Recommendation: [proceed / review with human / reduce scope]
```

If scope IS explicitly L (user or productowner pre-approved): log the report but do NOT halt.

### 11. Implement

Run `/speckit.implement`.

### 12. E2E Tests (conditional)

**Only if** ALL of these are true:
- PLAYWRIGHT_CONFIG exists
- E2E_SKILL is available
- `plan.md` references frontend files (e.g., files under `client/src/`, `src/pages/`, `src/components/`)

Then run `/speckit.e2e`.

If E2E writes `blockers.md`, check the `failure_class` field:

- **`spec-ambiguity`**: Re-run `/speckit.specify` with the `respec_request` from blockers.md, then restart from Step 7 (Plan). Cap at **one round-trip** — if the same class recurs, halt.
- **`plan-gap`**: Re-run `/speckit.plan` with the `replan_request`, then restart from Step 8 (Tasks). Cap at **one round-trip**.
- **`task-decomposition` / `implementation-bug` / `infra`**: Standard halt — write blockers.md and stop.

**Skip** if any condition is false (not an error — just skip silently).

### 12.5. AI Code Review (skip if XS)

Run `/speckit.review`.

If review reports any FAIL scores: read `{FEATURE_DIR}/blockers.md` and **halt** for human review.
If review reports only PASS/WARN: continue to commit.

### 13. Commit and PR (if --commit flag)

1. Use the `/commit` skill to create the git commit. Do NOT create commits manually.
2. Push the branch: `git push -u origin HEAD`
3. Create PR: `gh pr create --fill`

If `/commit` skill is not available, fall back to manual `git add` + `git commit`.

### 13.5. Collect Pipeline Metrics

Write `{FEATURE_DIR}/pipeline-metrics.json` with timing and stats for each stage that ran:

```json
{
  "feature_id": "{FEATURE_ID}",
  "scope": "{XS|S|M|L}",
  "stages": {
    "specify": {"ran": true, "files_created": ["spec.md"]},
    "clarify": {"ran": true, "decisions_count": 5},
    "quality_gate": {"ran": true, "iterations": 1, "all_pass": true},
    "plan": {"ran": true, "files_created": ["plan.md", "research.md", "data-model.md"]},
    "tasks": {"ran": true, "task_count": 18, "parallel_count": 6},
    "implement": {"ran": true, "files_modified": 14, "tasks_completed": 18, "tasks_flagged": 0},
    "e2e": {"ran": false, "reason": "backend-only"},
    "review": {"ran": true, "pass": 5, "warn": 1, "fail": 0},
    "commit": {"ran": true, "pr_url": "..."}
  },
  "fix_iterations": 0,
  "halted": false,
  "ts": "YYYY-MM-DDTHH:MM:SSZ"
}
```

Only include stages that actually ran. Omit stages that were skipped due to scope.

### 14. Report

Display a summary of what was done:

```
## Pipeline Complete: {feature-description}
## Scope: {scope} (XS|S|M|L)

### Steps Executed
- [x] Specify — spec.md created  [or: skipped — XS scope]
- [x] Self-clarification — decisions.md ({N} decisions)  [or: skipped — no autonomous constitution]
- [x] Quality gate — all items PASS  [or: skipped — no quality-gate.md]
- [x] Plan — plan.md created
- [x] Tasks — tasks.md created ({N} tasks)
- [x] Task validation — all rules pass  [or: skipped]
- [x] Pre-flight — all assertions pass  [or: skipped]
- [x] Implement — all tasks executed
- [x] E2E — tests pass  [or: skipped — no playwright config / backend-only]
- [x] Review — N/6 PASS, N/6 WARN  [or: skipped — XS scope]
- [x] Metrics — pipeline-metrics.json written
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
