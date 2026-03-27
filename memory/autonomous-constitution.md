## Autonomous Clarification Protocol

After every `/speckit.specify` run, before proceeding to planning, the agent MUST:

1. Re-read the generated `spec.md` in full.
2. Adopt the role of a skeptical Product Manager and identify all requirements that are:
   - Ambiguous (could be interpreted in more than one way)
   - Missing (implied by context but not stated)
   - Contradictory (two requirements conflict with each other)
3. For each issue found, resolve it using the most conservative and safe interpretation.
4. Document every resolution in `.specify/specs/<feature>/decisions.md` using this format:

   ```
   ### Decision: <short title>
   **Issue:** <what was ambiguous/missing/contradictory>
   **Resolution:** <what was decided>
   **Rationale:** <why this interpretation was chosen>
   ```

5. Do NOT ask for human input. Do NOT pause. Resolve all issues autonomously and document them.
6. Only proceed to `/speckit.plan` once `decisions.md` has been written and all open issues are resolved.

## Spec Quality Gate

After writing `decisions.md` and before running `/speckit.plan`, the agent MUST run `/speckit.checklist` against the criteria in `.specify/memory/quality-gate.md`.

For each checklist item:
- Mark PASS or FAIL with a one-line rationale.
- If any item FAILS: fix `spec.md` immediately, then re-run the checklist.
- Repeat until all items PASS.
- Do NOT proceed to planning with any failing items.
- Do NOT ask for human review. Fix and re-score autonomously.

Record the final checklist scores in `.specify/specs/<feature>/quality-report.md`.

## Task Structural Validation

After running `/speckit.tasks` and before running `/speckit.implement`, the agent MUST validate `tasks.md` against all of the following rules:

**Coverage:**
- Every user story in `spec.md` maps to at least one task in `tasks.md`.
- Every acceptance criterion in `spec.md` is addressed by at least one task.

**Structure:**
- Every task specifies at least one target file path.
- Every task has a clear, unambiguous success condition.
- No task contains open questions or unresolved references.

**Ordering:**
- No task has a dependency on a task that appears after it in the list.
- Tasks marked `[P]` (parallel) do not share write targets with each other.

**Autonomy check:**
- Read every task and ask: "Can this task be implemented without human input?"
- If any task requires human input to proceed, either resolve the blocker using available context or add it to a `blockers.md` file and halt (notify human).

If any validation rule fails: fix `tasks.md` and re-validate. Do not proceed with a broken task list.

## Pre-Flight Assertions Before Implementation

Before executing `/speckit.implement`, assert ALL of the following. If any assertion fails, halt and report — do NOT proceed:

- [ ] `constitution.md` exists and was referenced during spec and planning phases.
- [ ] `spec.md` exists and has no unchecked checklist items.
- [ ] `decisions.md` exists and documents all assumption resolutions.
- [ ] `quality-report.md` exists and shows all items PASSING.
- [ ] `tasks.md` passed all structural validations (coverage, structure, ordering, autonomy).
- [ ] No `blockers.md` exists with unresolved items.

If all assertions pass: proceed with `/speckit.implement` without waiting for human confirmation.
If any assertion fails: create or update `blockers.md` with the specific failure, then halt and notify.

## E2E Validation Protocol

After `/speckit.implement` and before committing:

1. Run `/speckit.e2e` to generate and run E2E tests from `spec.md` acceptance criteria.
2. Tests use existing project test infrastructure if available.
3. Reuse page objects and helpers — only extend, don't duplicate.
4. 3 fix-and-retry cycles max — then write `blockers.md` and halt.
5. Skip for backend-only features (no frontend files in `plan.md`).
