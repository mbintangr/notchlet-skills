---
name: notchlet-deliver
description: Use when delivering implemented Notchlet work. Opens the PR from the feature branch, runs review against the backlog description, drives the fix-commit-push loop, merges the PR, records development evidence, sets the backlog item to DONE, and syncs the Git issue when authorized.
---

# Deliver Notchlet work

Delivery starts when the implementation branch is pushed and ends when the merged PR is recorded and the backlog item is `DONE`. Notchlet remains the source of status; the repository is the source of truth.

## Capability mode (check first)

- **Connected** — authenticated MCP tools are advertised, or `notchlet doctor` reports a workspace binding. Record remote evidence, set `DONE`, and sync the Git issue on the platform.
- **Local** — no account/platform MCP, but the CLI or offline server is available. Deliver through Git where a remote exists, record local evidence, and close locally; remote `DONE` and issue sync stay `sync pending`.
- **Neither** — deliver in the repository only and report that tracking is unavailable.

Prefer connected whenever present; never silently fall back to local. Never present local closure as a remote status update.

## Entry criteria

- Implementation is complete and pushed via `notchlet-implement`.
- The feature branch exists and is up to date with the target branch (`main`/`develop`).
- The item is resolved and set to `IN_PROGRESS` (move to `IN_REVIEW` when the PR opens).
- The user has authorized opening the PR. If no Git remote or PR exists, delivery ends at local evidence and `change close`.

## Step 1 — Open the PR

1. Open a PR/MR from the feature branch into the target branch.
2. Title and body must reference the backlog key and link the Git issue. Summarize scope and list the acceptance criteria being met.
3. Verify the PR number/URL actually returned by the provider; never invent one.
4. Set the backlog item to `IN_REVIEW`.

Confirm before opening the PR. Do not push unrelated commits or include secrets. If there is no Git remote or PR provider, skip to Step 5 and record local evidence with `change close`.

## Step 2 — Review against the backlog description

Run `notchlet-review` using the latest backlog description and acceptance criteria. Distinguish a self-recorded check from independent human review.

## Step 3 — Fix loop (if changes are requested)

1. Return to `notchlet-implement` and fix every requested item.
2. Run focused checks again.
3. Commit and push to the same branch (authorized).
4. Re-run `notchlet-review` on the changed scope.

Repeat until review approves. Do not merge an unapproved PR when review is required by policy.

## Step 4 — Merge

1. Merge the PR only when authorization is explicit.
2. Verify the PR state is actually merged. A closed-unmerged PR, a local commit, or issue closure is not merge proof.
3. Confirm the target branch contains the change.

## Step 5 — Evidence and DONE

### Definition of Done

- [ ] Every acceptance criterion is met with actual evidence (test output, diff, command results).
- [ ] All focused checks pass.
- [ ] Review is approved, or review is not required per team policy.
- [ ] The PR is verified merged.
- [ ] The parent item is not marked DONE while required children remain open.
- [ ] No unresolved blockers remain.

### Record evidence

Add one `### Development evidence` comment to the backlog item:

```markdown
## Development evidence — <date>
- Backlog item: <key and title>
- Scope delivered: <behavior>
- Changed components: <paths>
- Verification: <exact commands, working directories, outcomes>
- Acceptance criteria: <met / remaining, with evidence>
- Git evidence: <actual commit SHA and PR URL>
- Blockers or follow-up: <facts, or none>
- Resulting status: <status and reason>
```

Do not publish credentials, personal payloads, or raw production data.

### Update status and sync

- Connected: set the item to `DONE` and confirm the returned state.
- Local: record local evidence, run `notchlet change close <change> --preview`, and close when eligible; report remote `DONE` as `sync pending`.
- Sync the Git issue when authorized. Ticket `RESOLVED` is distinct from `CLOSED`; do not auto-close during delivery.
- Report partial failures without claiming all systems were updated.

**Mode behavior:** connected — remote evidence, `DONE`, and issue sync. Local — local evidence and local closure; remote status and issue sync remain `sync pending` until connectivity returns, then run `notchlet sync status` and sync when authorized.

## Rollback and regression handling

- Do not mark DONE if delivery causes a regression.
- Report the regression immediately, create a follow-up item via `notchlet-plan`, and set the affected item back to `IN_PROGRESS` or `IN_REVIEW`.
- Watch builds, tests, and production errors for a reasonable period after delivery.

## CLI tracking layer (required when available)

The CLI holds the final verification and evidence record:

- `notchlet verify <change> --record` and `notchlet evidence list --change <change>`.
- Connected: `notchlet change close <change> --preview`, then `notchlet sync push --change <change>` when authorized.
- Local: `notchlet change close <change> --preview` and close when eligible; sync later.
- `notchlet checkpoint --change <id> --name <name>` for a synchronized checkpoint; it is not delivery evidence by itself.

Local closure, remote delivery, PR merge, and deployment are separate outcomes. Do not claim a remote backlog update occurred when only local evidence was recorded. Full command reference: `notchlet-super`.

## When NOT to use

- To plan — use `notchlet-plan`.
- To implement or fix code — use `notchlet-implement`.
- To review only — use `notchlet-review`.
- To set up the MCP connection — use `notchlet-mcp`.

## Related skills

- `notchlet-implement` — implements the changes and pushes the branch
- `notchlet-review` — reviews the PR against the backlog description
- `notchlet-platform` — reads the backlog item and context for delivery
- `notchlet-super` — orchestrates the full cycle and holds the CLI command reference
