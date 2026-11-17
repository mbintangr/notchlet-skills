---
name: notchlet-review
description: Use when reviewing implemented Notchlet backlog work, approving or requesting changes on PRs/MRs, and handling review feedback before delivery. Sits between notchlet-implement and notchlet-deliver in the development cycle.
---

# Notchlet Code Review

Review implemented work against acceptance criteria before it is delivered. A backlog item should not move to `DONE` without passing review.

## Entry criteria

- Implementation is complete and committed on a feature branch.
- PR/MR is open against the target branch (main/develop).
- `notchlet-implement` has been run and the item is set to `IN_REVIEW` or ready for review.
- All focused checks (lint, type, tests) have passed locally.

## Review checklist

### Requirements coverage
- [ ] Every acceptance criterion from the backlog item is addressed by the implementation.
- [ ] No scope creep: nothing implemented that was not requested or accepted.
- [ ] Edge cases and failure behavior are handled (timeouts, validation errors, unauthorized access).

### Code quality
- [ ] No exposed secrets, credentials, or personal data in the diff.
- [ ] No unintended files (build artifacts, node_modules, .env, local config) in the diff.
- [ ] Existing abstractions reused where appropriate; no unnecessary new abstractions.
- [ ] Authorization, validation, and caller integration are present where required.

### Testing
- [ ] Focused checks suited to the change have passed (lint, type-check, unit tests).
- [ ] Test output is actual, not invented or assumed.
- [ ] Pre-existing failures are distinguished from failures caused by the change.

### Backlog consistency
- [ ] The backlog item description matches the implemented behavior.
- [ ] `expectedDescriptionRevision` was used on any description edits; conflicts reconciled.
- [ ] Checkpoint comments recorded on the backlog item with approach, changed files, and check outcomes.

## Approve or request changes

- **Approve:** If all checklist items pass. Set the backlog item to `IN_REVIEW` (or keep it there) and notify the implementer that delivery can proceed via `notchlet-deliver`.
- **Request changes:** If any checklist item fails. Add a specific comment on the PR/MR and backlog item listing what must be fixed. Do not set the item to `DONE`. Return to `notchlet-implement` until resolved.

## Handle review feedback

- If the implementer requests clarification or disputes a finding, discuss on the backlog item comment thread, not in private messages.
- If changes are required, the implementer updates the code, re-runs checks, and re-submits for review.
- Re-review only the changed files and the specific feedback items, not the entire implementation from scratch.

## When to use

- After `notchlet-implement` completes and before `notchlet-deliver`.
- When a PR/MR needs approval or review feedback.
- When reviewing another agent's or human's implementation against a Notchlet backlog item.

## When NOT to use

- For initial planning (use `notchlet-plan`).
- For writing implementation code (use `notchlet-implement`).
- For final status sync to Notchlet (use `notchlet-deliver`).
