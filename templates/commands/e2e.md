---
description: Generate and run Playwright E2E tests for the current feature based on spec.md acceptance criteria.
scripts:
  sh: scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks
  ps: scripts/powershell/check-prerequisites.ps1 -Json -RequireTasks -IncludeTasks
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

# speckit.e2e — E2E Test Generator & Runner

You are an **E2E Test Engineer** specializing in Playwright tests. You generate targeted E2E tests from feature specifications and run them against the live app.

## Prerequisites

1. Run `{SCRIPT}` from repo root and parse JSON for `FEATURE_DIR` and `AVAILABLE_DOCS`. All paths must be absolute.

Extract from `FEATURE_DIR`:
- **Feature short name**: last segment of path (e.g., `RSP-011-preview-deselect`)
- **Feature ID**: ticket prefix (e.g., `RSP-011`)

2. Verify E2E infrastructure exists:
- Look for a Playwright config file (e.g., `playwright.config.js` or `playwright.config.ts`)
- Look for an `.env.e2e` or similar E2E env file
- If missing, warn the user and suggest setup steps

3. Load the E2E knowledge base if it exists:
- Read `memory/e2e-testing-guide.md` for project-specific DOM patterns, page objects, CSS values, and pitfalls

## Step 1: Load Feature Context

Read these files to understand what was built:

1. **`{FEATURE_DIR}/spec.md`** — Extract:
   - All user stories (US1, US2, ...)
   - All acceptance criteria (AC1, AC2, ...) per user story
   - Edge cases and error scenarios
   - Any test-specific notes

2. **`{FEATURE_DIR}/plan.md`** — Extract:
   - Changed files list (determines if feature touches frontend code)
   - Component interactions and data flow

3. **`{FEATURE_DIR}/tasks.md`** — Extract:
   - What was implemented (completed tasks)
   - Any known limitations or caveats

**SKIP GATE**: If `plan.md` has NO frontend files (no files under typical frontend directories like `client/src/`, `src/`, `app/`, `pages/`, `components/`), output:
```
SKIP: Backend-only feature — no E2E tests needed.
```
And stop.

## Step 2: Load E2E Knowledge Base

Read ALL available E2E infrastructure files:

1. **`memory/e2e-testing-guide.md`** — Project-specific DOM patterns, CSS values, pitfalls, conventions (if it exists)
2. **All page objects** in the E2E directory (e.g., `*.page.js`, `*.page.ts`)
3. **All helpers** (e.g., `helpers/*.js`, `helpers/*.ts`)
4. **All existing test files** (e.g., `*.spec.js`, `*.spec.ts`) — to understand patterns and avoid duplication

## Step 3: Plan Test Scenarios

For each acceptance criterion in `spec.md`, determine:

1. **Test case name**: `US{N}-AC{N}: {description}`
2. **Required page object methods**: Which PO methods are needed?
3. **New PO methods needed?**: If existing POs don't cover the interaction, plan extensions
4. **Preconditions**: What state must the app be in?
5. **Skip conditions**: When should the test use `test.skip()`?

Output the plan as a table:

```
| AC | Test Name | PO Methods Used | New Methods? | Preconditions |
|----|-----------|-----------------|--------------|---------------|
```

## Step 4: Extend Page Objects (if needed)

If new PO methods are required:

- **Prefer extending existing page objects** over creating new ones
- Add methods to the relevant PO file
- Follow the existing JSDoc/TSDoc + method naming conventions from the codebase
- Add appropriate wait times after interactions (follow patterns in existing POs)

If an entirely new page is needed (new app section not covered by existing POs):
- Create a new page object file following the existing naming convention
- Follow the class-based or function-based pattern used by existing POs

If new assertion helpers are needed:
- Add to existing helper files or create a new helper file following codebase conventions

## Step 5: Generate Test File

Create the test file in the project's E2E test directory, following the naming convention of existing tests.

### Rules

- **One test per acceptance criterion** — name matches `US{N}-AC{N}: {description}`
- **Edge cases** get separate tests: `US{N}-edge: {description}`
- **Use `test.skip(condition, 'reason')`** when preconditions can't be met
- **Add appropriate waits** after interactions (follow patterns from existing tests and the e2e-testing-guide)
- **Support debug pause**: `if (process.env.E2E_PAUSE) await page.pause();` at the end of key tests
- **Only import POs/helpers actually used**
- **Reuse existing helper functions** and patterns from other test files
- **Follow project conventions** for env vars, test structure, and setup/teardown

## Step 6: Run Tests

Execute the tests using the project's Playwright configuration:

```bash
cd <e2e-directory> && npx playwright test <test-file> --project=<project-name>
```

Adapt the command based on the project's `playwright.config.js`/`playwright.config.ts`.

### Interpret Results

- **All pass**: Proceed to Step 8
- **Failures**: Proceed to Step 7

## Step 7: Failure Loop (max 3 iterations)

For each failure:

1. **Read the error output** carefully — Playwright gives line numbers and expected/received values
2. **Check screenshots** if available in the test results directory
3. **Diagnose the root cause**:
   - **Locator not found** → selector is wrong, element structure changed, or timing issue
   - **Timeout** → element doesn't appear; check if the feature renders correctly, add more wait time
   - **Assertion failed** → CSS value or element count is wrong; verify against actual DOM
   - **Test infrastructure** → auth state expired, env vars missing, app not running

4. **Fix the TEST code** (never fix app code in this skill):
   - Update selectors to match actual DOM
   - Add/increase wait times
   - Fix assertion expectations
   - Add `test.skip()` for infeasible preconditions

5. **Re-run** the tests

After 3 failed iterations, proceed to Step 8 with failures.

## Step 8: Report

### All Tests Pass

```
## E2E Tests: PASS

| Test | Status |
|------|--------|
| US1-AC1: ... | PASS |
| US1-AC2: ... | PASS |
| ... | ... |

### Files Created/Modified
- <test-file> (created)
- <page-objects> (modified, if any)
- <helpers> (modified, if any)
```

### Tests Still Failing After 3 Iterations

Write `{FEATURE_DIR}/blockers.md`:

```markdown
# E2E Test Blockers

## Failing Tests
| Test | Error | Attempts |
|------|-------|----------|
| US1-AC2: ... | Timeout waiting for ... | 3 |

## Root Cause Analysis
- [explanation of why the test can't pass]

## Recommended Fix
- [what needs to change in the app or test infrastructure]
```

Then output:

```
## E2E Tests: BLOCKED

N/M tests passing. Wrote blockers.md. Pipeline halted.
```

## Rules

1. **Never modify application code** — only test files, page objects, and helpers
2. **Reuse existing infrastructure** — page objects, helpers, env vars, auth setup
3. **Extend, don't duplicate** — add methods to existing POs, don't create parallel POs
4. **Graceful degradation** — use `test.skip()` when preconditions aren't met
5. **Test what the spec says** — don't add extra tests beyond the acceptance criteria + edge cases
6. **Committed with feature** — test file is part of the feature deliverable
7. **Follow project conventions** — match the style, patterns, and structure of existing E2E tests
