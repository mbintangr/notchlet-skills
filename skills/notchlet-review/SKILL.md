---
name: notchlet-review
description: Use when reviewing an open PR/MR for a Notchlet backlog item. Checks the implementation against the backlog description and acceptance criteria, approves, or requests changes and drives the fix-commit-push loop before merge.
---

# Review the PR against the backlog description

The backlog description is the review contract. Judge the PR against its acceptance criteria, not against the implementer's summary. A backlog item must not reach `DONE` without passing review.

## Capability mode (check first)

- **Connected** — authenticated MCP tools are advertised, or `notchlet doctor` reports a workspace binding. Review against the platform backlog description and acceptance criteria, and record platform review evidence.
- **Local** — no account/platform MCP, but the CLI or offline server is available. Review against the local change requirements and checks; remote status and issue updates stay `sync pending`.
- **Neither** — review against the user's stated requirements and report that tracking is unavailable.

Prefer connected whenever present; never silently fall back to local. A local approval is not a remote status update.

## Entry criteria

- Implementation is pushed on a feature branch and a PR/MR is open against the target branch (`main`/`develop`), or the work is a local-only delivery without a remote PR.
- The requirement source is read: the platform backlog description in connected mode, or the local change requirements in local mode.
- Focused checks (lint, type, tests) have been run on the branch.

If no PR exists yet, use `notchlet-deliver` to open it first.

## Review against the backlog description

### Requirements coverage
- [ ] Every acceptance criterion in the backlog description is demonstrably addressed.
- [ ] The PR title/body references the backlog key and links the Git issue.
- [ ] No scope creep: nothing implemented that was not requested or accepted.
- [ ] Edge cases and failure behavior match the description (timeouts, validation errors, unauthorized access, empty state).

### Code quality
- [ ] No exposed secrets, credentials, or personal data in the diff.
- [ ] No unintended files (build artifacts, `node_modules`, `.env`, local config) in the diff.
- [ ] Existing abstractions are reused; no unnecessary new abstractions.
- [ ] Authorization, validation, and caller integration are present where required.

### Testing
- [ ] Focused checks pass and the output is actual, not invented.
- [ ] Pre-existing failures are distinguished from failures caused by the change.
- [ ] Verification evidence matches the criteria's verification section.

### Backlog consistency
- [ ] The implemented behavior matches the description; discrepancies are flagged.
- [ ] Any description edit used `expectedDescriptionRevision` and reconciled conflicts.
- [ ] A checkpoint comment records approach, changed files, and check outcomes.

## Approve or request changes

**Approve** when every item above passes:

- Add an approval comment on the PR linking the backlog key.
- Connected: leave the item in `IN_REVIEW`. Local: keep the local state and mark remote status `sync pending`.
- Hand off to `notchlet-deliver` to merge.

**Request changes** when any item fails:

- List specific, actionable fixes on the PR and on the backlog item. One comment per finding is fine; avoid vague feedback.
- Do not set the item to `DONE` and do not merge.
- Return to `notchlet-implement`: fix, run checks, commit, push.
- Re-review only the changed scope and the requested fixes, not the whole implementation from scratch.

## Handle discussion

- Discuss findings on the PR or backlog comment thread, not in private side channels.
- If the implementer disputes a finding, resolve it against the backlog description and acceptance criteria.
- If the description itself was wrong, revise it through `notchlet-plan` before re-review.

## CLI tracking layer (required when available)

Record review evidence against the local change's checks (`notchlet verify <change> --record`, `notchlet evidence list --change <change>`). Connected: this evidence is linked to the backlog item. Local: it is the primary record until sync.

Local verification is not independent human review, and it is not a remote backlog update. Full command reference: `notchlet-super`.

## When NOT to use

- To plan work — use `notchlet-plan`.
- To write implementation code — use `notchlet-implement`.
- To open the PR or merge — use `notchlet-deliver`.

## Related skills

- `notchlet-implement` — implements fixes requested during review
- `notchlet-deliver` — opens the PR, invokes review, merges, and records evidence
- `notchlet-platform` — reads the backlog description and context for review
- `notchlet-super` — orchestrates the full cycle and holds the CLI command reference
