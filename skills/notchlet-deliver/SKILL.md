---
name: notchlet-deliver
description: Use when recording development evidence, reviewing completed Notchlet work, synchronizing authorized Git issues, or updating backlog and ticket status after implementation and verification. Final step before work is considered complete.
---

# Evidence-based Notchlet delivery

Native `backlog_comment_add` and related context tools are available on the connected `notchlet_*` host; discover live tools and use supported fallbacks. Use `expectedDescriptionRevision` on description edits when supported, reconciling conflicts by rereading.

## Entry criteria

- Implementation is complete via `notchlet-implement`.
- Code review via `notchlet-review` is approved (or review is not required per team policy).
- All focused checks pass (lint, type-check, tests).
- PR/MR is merged (for branch-linked work) or implementation is committed (for issue-only items).

## Evaluate readiness

- Compare implementation and checks against every acceptance criterion.
- Inspect actual test output and diff. Record failures and blockers rather than treating an attempted test as a passing check.
- For branch-linked coding work, verify the matching PR/MR is confirmed merged; issue closure, a closed-unmerged PR/MR, or a local commit alone is not merge proof. Do not report branch-linked work complete until merge is verified. Issue-only items can retain their existing acceptance policy.
- Use `IN_PROGRESS` for incomplete implementation or unresolved verification blockers.
- Use `IN_REVIEW` when implementation/checks are ready and required review or acceptance remains.
- Use `DONE` only when the item's completion conditions are met and required review has occurred. Do not treat a PR, commit, or successful build alone as proof of feature completion.
- An epic/story is complete only when its own acceptance criteria and all required children are complete. Re-read the hierarchy before updating a parent.

## Definition of Done (checklist before marking DONE)

- [ ] Every acceptance criterion is met with actual evidence (test output, diff, command results).
- [ ] All focused checks pass (lint, type-check, tests).
- [ ] Code review is approved (via `notchlet-review`) or review is not required.
- [ ] PR/MR is merged (branch-linked work) or implementation is committed (issue-only work).
- [ ] Backlog item status reflects the true state (`IN_PROGRESS`, `IN_REVIEW`, or `DONE`).
- [ ] Parent item is not marked `DONE` while required children remain open.
- [ ] No unresolved blockers remain; blockers are documented if present.

## Persist delivery evidence

Record a meaningful checkpoint and final verification in an ordinary backlog comment when comment tools are available, preserving the description. Prefer one concise `### Agent checkpoint` for approach/progress, changed files, exact checks/results, verified branch/commit, blockers, and next action; avoid per-tool-call comments. If updating a short approach in the description, use the latest `expectedDescriptionRevision` when supported and reconcile conflicts. For substantial reports, save a CUSTOM draft document and reference its title from the item. Do not create a report document on every retry.

```markdown
## Development evidence — <date>
- Scope delivered: <behavior>
- Changed components: <paths>
- Verification: <exact commands, working directories, outcomes>
- Acceptance criteria: <met / remaining, with evidence>
- Git evidence: <actual commit/PR URL if available>
- Blockers or follow-up: <facts, or none>
- Resulting status: <status and reason>
```

Do not publish credentials, personal payloads, or raw production data in reports.

## Synchronize

- Update the selected backlog item's status and confirm the returned state.
- Do not complete a parent while required child items remain open; verify linked ticket synchronization rather than blindly forcing a ticket status.
- Change a linked ticket only if the requested scope includes it and the ticket's resolution criteria are satisfied. Ticket `RESOLVED` is distinct from `CLOSED`; do not auto-close during coding.
- Create/link/sync remote issues only when authorized. Verify the remote result. Report partial failures without claiming all systems were updated.
- Do not invent comments, assignment, dependencies, or PR-management tools that are absent from the live schema.
- End with a self-contained summary of changes, verification, Notchlet status, and unresolved work.

## Post-delivery monitoring

- Watch for build failures, test failures, or production errors linked to the delivered item for a reasonable period after delivery.
- If the delivered item causes a regression, report it immediately and use `notchlet-implement` to create a follow-up fix item.
- Do not assume delivery is permanent success; verify the change behaves as expected in the target environment.

## Rollback procedure (if delivery breaks production)

- Do not mark the item `DONE` if it causes a regression.
- Create a follow-up backlog item for the fix via `notchlet-plan`.
- Set the affected item back to `IN_PROGRESS` or `IN_REVIEW` as appropriate.
- Document the regression evidence in a backlog comment.

## When to use

- Final step after `notchlet-implement` and `notchlet-review`.
- Recording evidence, updating status, and synchronizing Git issues/tickets.
- When work is ready to be considered complete.

## When NOT to use

- When you only need to plan tasks — use `notchlet-plan` instead.
- When you only need to implement code — use `notchlet-implement` instead.
- When you only need to review — use `notchlet-review` instead.
- When you need to set up the MCP connection first — use `notchlet-mcp` instead.

## Related skills

- `notchlet-implement` — implements work before delivery
- `notchlet-review` — reviews implementation before delivery
- `notchlet-platform` — resolves backlog items and reads context for delivery
- `notchlet-super` — orchestrates the full cycle including delivery
