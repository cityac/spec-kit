# Spec Quality Gate

This checklist is used by the agent to self-score every specification before planning begins.
All items must PASS before proceeding. Fix and re-score if any item fails.

## Completeness

- [ ] Every feature mentioned in the prompt has at least one user story.
- [ ] Every user story has at least one measurable acceptance criterion (not vague like "works correctly").
- [ ] All happy-path flows are fully described end-to-end.
- [ ] At least the most likely edge cases and error states are enumerated per user story.
- [ ] No user story uses "should" without an explicit rationale for why it is optional.

## Clarity

- [ ] No requirement can be interpreted in more than one way without an explicit decision documented in `decisions.md`.
- [ ] All referenced entities (users, roles, objects, states) are defined in the spec or in a glossary.
- [ ] No requirement references external documents or context that is not available in the repo.

## Consistency

- [ ] No two requirements contradict each other.
- [ ] Terminology is consistent throughout the spec (no synonyms for the same concept).
- [ ] User stories are consistent with the governing principles in `constitution.md`.

## Implementability

- [ ] Every acceptance criterion is verifiable by a test or a deterministic check.
- [ ] No acceptance criterion requires subjective human judgment to evaluate.
- [ ] The spec does not prescribe implementation details that belong in the plan, not the spec.

## API & Integration (if applicable)

- [ ] Every API interaction specifies both success responses and error states.
- [ ] No API contract leaves authentication, authorization, or rate limiting undefined if applicable.
