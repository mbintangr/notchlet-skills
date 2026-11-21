---
name: notchlet-super
description: Use when you want to run the complete Notchlet-driven development cycle end-to-end — grill and plan, create backlogs, create the Git issue and branch, develop, push, open a PR, review it against the backlog description, fix, merge, and mark done — with authorization gates at every external side effect. Resume valid in-progress work instead of restarting completed phases.
---

# Notchlet Super — full-cycle orchestrator

Drive work from idea to merged using the Notchlet platform for requirements and status, Git for implementation truth, and the Notchlet CLI as the primary local tracking layer.

## Capability mode (check first)

Decide the mode once per session and reuse it unless connectivity changes:

- **Connected** — authenticated MCP tools are advertised, or `notchlet doctor` reports a workspace binding. The platform is the authority for requirements and status; the CLI is the primary local tracking layer.
- **Local** — no account/platform MCP, but the CLI or offline server is available. The CLI is the primary workflow for the plan, tasks, and evidence; platform-only steps are deferred as `sync pending`.
- **Neither** — do repository-only work and report that tracking is unavailable.

Prefer connected whenever it is present. Never silently fall back to local because local always works. Never present local evidence, local closure, or a local status as a remote update. When connectivity returns, run `notchlet sync status --change <change>`, reconcile, and sync only when authorized.

## The base flow

| Step | Phase | Agent action | Authorization gate |
|------|-------|--------------|--------------------|
| 1 | Plan & grill | Interrogate the request until requirements are unambiguous, then produce a plan. | — |
| 2 | Create backlogs | Create epic → stories → tasks on Notchlet with a Definition of Ready. | Confirm the backlog plan |
| 3 | Implement | Create/link the Git issue, create the branch from the item, checkout locally, develop, run checks, commit, push. | Confirm issue + branch; confirm push |
| 4 | Deliver | Open a PR, review it against the backlog description, fix/commit/push if rejected, merge. | Confirm PR; confirm merge |
| 5 | Done | Record evidence, set the backlog item to DONE, sync the Git issue. | Confirm DONE |

Never combine gates silently. If the user pre-authorizes a group of actions, that authorization applies only to the stated scope.

Git steps (issue, branch, commit, push, PR, merge) run in both modes. Platform steps (backlog create/verify, Notchlet issue linking, remote status, `DONE`) are required in connected mode and deferred as `sync pending` in local mode.

## Phase 1 — Plan & grill (`notchlet-plan`)

Ask before writing anything:

- What outcome must be observable when this is done?
- Who uses it, and what is the primary path?
- What is explicitly out of scope?
- What are the failure modes and edge cases?
- Does it touch data, migrations, integrations, or permissions?
- How will we verify it (tests, commands, evidence)?
- What dependencies or blockers exist?

Ask in small batches, one decision per question, and stop when the Definition of Ready can be met. Summarize the answers back to the user before creating backlog items.

**Proceed when:** the plan is confirmed.

## Phase 2 — Create backlogs (`notchlet-plan`)

- Resolve the product; check for duplicate items and refine instead of duplicating.
- Create epic → stories → tasks sequentially, capturing each returned numeric ID for `parentId`.
- Apply the Definition of Ready: measurable acceptance criteria, dependencies (or "none"), specific scope, verification criteria, no blocking dependency.
- Use the description template: Outcome / Scope / Acceptance criteria / Dependencies / Verification.
- Verify the created hierarchy by reading it back.

**Mode behavior:** connected — create and verify the hierarchy on the platform, then import the item with `notchlet backlog import` and author the executable checks/tasks locally. Local — author the plan as a local change and record platform backlog creation as `sync pending`.

**Proceed when:** the backlog hierarchy is verified and the first executable item is ready (or, in local mode, the local change plan is validated).

## Phase 3 — Implement (`notchlet-implement`)

For the selected backlog item:

1. **Create the Git issue** with `notchlet_git_issue_create` (or `notchlet_git_issue_link` for an existing issue). Verify the returned issue number/URL.
2. **Create the branch** as `<type>/<backlog-key>-<slug>` (for example `feat/APP-42-slack-toggle`) from an up-to-date `main`/`develop`.
3. **Checkout the branch locally.**
4. Set the item to `IN_PROGRESS`.
5. Implement the complete behavior, including authorization, validation, failure handling, and caller integration.
6. Run focused checks: lint, type-check, unit tests, and integration tests where relevant.
7. Commit with a conventional message referencing the backlog key.
8. Push when authorized.
9. Add an `### Agent checkpoint` comment with approach, changed files, exact check results, branch/commit, blockers, and next action.

**Mode behavior:** connected — create/link the issue with the Notchlet Git issue tools, import the item to a local change, and use `task start/complete` plus `verify --record` as you go. Local — create the provider issue if a provider CLI is available (link it later), use the local change for tasks/verify, and mark the Notchlet issue link `sync pending`.

**Proceed when:** implementation is pushed (or the user chooses to stop at the local commit).

Do not open a PR here; that belongs to delivery.

## Phase 4 — Deliver (`notchlet-deliver`, review via `notchlet-review`)

1. **Open the PR** from the feature branch into the target branch. Reference the backlog key in the title and body and link the Git issue.
2. **Review against the backlog description** and every acceptance criterion using `notchlet-review`.
3. **If changes are requested:** return to implementation, fix, commit, push, and re-review the changed scope.
4. **Merge the PR** when authorized.
5. Verify the PR actually merged; a closed-unmerged PR is not merge proof.

**Mode behavior:** connected — record review evidence and set the item to `IN_REVIEW` through the platform. Local — review against the local change requirements and record evidence locally; remote status stays `sync pending`. Local without a Git remote: skip PR/merge and deliver locally.

Do not deploy or publish as part of this step unless separately authorized.

## Phase 5 — Done (`notchlet-deliver`)

- Check the Definition of Done: every acceptance criterion has evidence, checks pass, review approved, PR merged, no blockers, and no open required children.
- Record a `### Development evidence` comment.
- Set the item to `DONE` and confirm the returned state.
- Sync the Git issue when authorized. `RESOLVED` is not `CLOSED`.
- Watch for regressions; create a follow-up item if delivery breaks something.

**Mode behavior:** connected — record remote evidence, set `DONE`, and sync the issue. Local — record local evidence, run `notchlet change close --preview` (then close when eligible), and report remote `DONE` and issue sync as `sync pending` until connectivity returns.

## Resume current work

Before starting any phase, inspect state and skip what is already valid:

- `git branch --show-current`, `git status --short`, recent commits.
- Existing backlog item, Git issue, open PR, and (if used) local change.
- Continue by state: no plan → plan; plan exists → implement; branch pushed → open PR; PR open → review; approved → merge; merged → evidence and DONE.

Reuse a valid MCP connection, workspace binding, product, repository, item, and branch. Do not restart setup, re-plan, or recreate issues/branches that already exist. If two records disagree (for example a local commit but no issue), reconcile rather than duplicating.

## CLI tracking layer (primary when available)

The CLI holds the executable plan, task progress, verification, and evidence. It is required whenever available: in connected mode it mirrors the platform backlog and syncs; in local mode it is the only tracking store until sync is possible. The Git steps still run in either mode.

- **Setup / binding:** `notchlet init`, `notchlet doctor`, `notchlet workspace bind`. Without an account, use `notchlet init --local --client opencode`.
- **Import the item (connected):** `notchlet backlog import <key-or-title>` from the initialized, correctly bound workspace. In local mode, create the change directly.
- **Author the plan:** `notchlet change update <change> ...`, `notchlet change add <change> requirement|check|scenario|decision|task ...`, or an atomic `notchlet change apply <change> --file <plan.json>`.
- **Tasks:** `notchlet task start|complete <task> --change <change>`.
- **Verify and record evidence:** `notchlet verify <change> --record`, then `notchlet evidence list --change <change>`.
- **Inspect state:** `notchlet status <change>`, `notchlet validate <change>`, `notchlet instructions <change> --action plan|implement|verify|resume|revise`.
- **Close and sync:** `notchlet change close <change> --preview`, `notchlet sync status|push|pull --change <change>`, `notchlet checkpoint --change <id> --name <name>`.

Rules: never hand-edit `tasks.json`; pass `--revision` on authoring writes so stale changes fail before writing; local closure, remote delivery, PR merge, and deployment are separate outcomes. In connected mode, sync changes back after delivery. Delivery evaluation inspects provider state; it does not merge or deploy.

## Safety

- [ ] Never push, open or merge a PR, deploy, or set DONE without explicit authorization. Create the remote issue and branch only behind the start-development gate.
- [ ] Prefer connected mode whenever platform MCP is available; never silently default to local-only.
- [ ] Never claim a tool succeeded when it returned an error or was unavailable.
- [ ] Never present local evidence, local closure, or local status as a remote update.
- [ ] Never fabricate check results, evidence, issue numbers, or PR URLs.
- [ ] Never write secrets or credentials into comments, branches, commits, or config.
- [ ] Never mark a parent DONE while required children remain open.
- [ ] Never treat a local commit, closed-unmerged PR, or `RESOLVED` ticket as proof of completion.

## Skills orchestrated

- `notchlet-mcp` — connection setup only when needed
- `notchlet-platform` — product and item resolution
- `notchlet-plan` — grilling, decomposition, Definition of Ready
- `notchlet-implement` — issue, branch, code, checks, commit, push
- `notchlet-review` — PR review against the backlog description
- `notchlet-deliver` — PR, merge, evidence, status, issue sync
