---
description: Run an AI code review on the current feature branch before creating a PR.
scripts:
  sh: scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks
  ps: scripts/powershell/check-prerequisites.ps1 -Json -RequireTasks -IncludeTasks
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

# speckit.review — AI Code Review

You are a **Senior Code Reviewer**. You review the implementation diff against the feature spec, constitution, and codebase patterns before the PR is created.

## Step 1: Setup

Run `{SCRIPT}` from repo root and parse JSON for `FEATURE_DIR` and `AVAILABLE_DOCS`. All paths must be absolute.

## Step 2: Load Context

Read these files to understand what was intended:

1. **`{FEATURE_DIR}/spec.md`** — what was specified (acceptance criteria)
2. **`{FEATURE_DIR}/plan.md`** — what was planned (architecture, file structure)
3. **`{FEATURE_DIR}/tasks.md`** — what tasks were defined
4. **`{FEATURE_DIR}/decisions.md`** — autonomous decisions made (if exists)
5. **`.specify/memory/constitution.md`** or **`.specify/memory/autonomous-constitution.md`** — project principles (if exists)

## Step 3: Get the Diff

Run `git diff main...HEAD` (or appropriate base branch) to get all changes in the feature branch.

If diff is large (>2000 lines), focus on:
- New files first
- Modified files with highest line count
- Skip generated files, lock files, and config-only changes

## Step 4: Review Checks

Run each check and score PASS / WARN / FAIL:

### 4a. Spec Compliance
- Every acceptance criterion in spec.md is addressed by the implementation
- No acceptance criteria left unimplemented
- Score: PASS if all addressed, WARN if minor gaps, FAIL if major gaps

### 4b. Constitution Compliance (if constitution exists)
- Diff doesn't violate any constitution principles
- Naming conventions followed
- Error handling patterns consistent
- Score: PASS / WARN / FAIL

### 4c. Pattern Consistency
- New code follows patterns established in existing codebase
- Imports, file structure, naming match project conventions
- No inconsistent patterns introduced (e.g., callbacks vs promises, different error handling)
- Score: PASS / WARN / FAIL

### 4d. Error Handling
- External calls have error handling
- User-facing errors have meaningful messages
- No swallowed errors (empty catch blocks)
- Score: PASS / WARN / FAIL

### 4e. Security
- No hardcoded secrets, tokens, or credentials
- User input validated/sanitized where applicable
- No SQL injection, XSS, or command injection vectors
- Auth checks present where needed
- Score: PASS / WARN / FAIL

### 4f. Scope Creep
- Files modified match what's in tasks.md
- No unexpected files changed that aren't in the plan
- Score: PASS / WARN if <3 extra files, FAIL if >5 extra files

## Step 5: Write Review Report

Write `{FEATURE_DIR}/review-report.md`:

```markdown
# Code Review: {feature-name}

## Summary
- Overall: PASS / WARN / FAIL
- Checks: N/6 PASS, N/6 WARN, N/6 FAIL

## Check Results
| Check | Score | Notes |
|-------|-------|-------|
| Spec Compliance | PASS | All 5 ACs addressed |
| Constitution | PASS | Follows all principles |
| Pattern Consistency | WARN | Mixed import style in utils/ |
| Error Handling | PASS | All external calls handled |
| Security | PASS | No issues found |
| Scope Creep | PASS | 0 unexpected files |

## Issues Found
### FAIL: [check name]
- **File**: path/to/file.ts:42
- **Issue**: [description]
- **Fix**: [suggestion]

### WARN: [check name]
- **File**: path/to/file.ts:15
- **Issue**: [description]
- **Suggestion**: [optional improvement]

## Files Reviewed
- [list of files in diff]
```

## Step 6: Report

Display review summary:

```
## Code Review: {status}

| Check | Score |
|-------|-------|
| Spec Compliance | ... |
| Constitution | ... |
| Patterns | ... |
| Error Handling | ... |
| Security | ... |
| Scope Creep | ... |

Issues: N FAIL, N WARN
Report: {FEATURE_DIR}/review-report.md
```

**If any FAIL**: The pipeline should halt for human review. Write issues to `{FEATURE_DIR}/blockers.md` with `failure_class: review-fail`.

**If only WARN or PASS**: Pipeline can continue.

## Rules

1. **Never modify application code** — only write the review report
2. **Be specific** — cite exact file paths and line numbers
3. **Don't be pedantic** — skip style nits that linters should catch
4. **Focus on correctness and security** — these matter most
5. **Compare against spec, not personal preference** — the spec is the source of truth
