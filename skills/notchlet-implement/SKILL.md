---
name: notchlet-implement
description: Use when implementing a Notchlet backlog item or ticket. Creates or links the Git issue, creates and checks out a branch from the item, implements the behavior, runs focused checks, commits, and pushes when authorized. The Notchlet CLI can add local change/task/verify tracking as an enhancement.
---

# Notchlet-driven development

Notchlet is the source of product requirements and status; the repository is the source of implementation truth. Implement the requested behavior, not just a ticket update.

## Capability mode (check first)

- **Connected** — authenticated MCP tools are advertised, or `notchlet doctor` reports a workspace binding. The platform is authoritative: create/link the issue with the Notchlet Git issue tools and mirror the work in a local change (`task start/complete`, `verify --record`).
- **Local** — no account/platform MCP, but the CLI or offline server is available. Use the local change for tasks, verification, and evidence; create the provider issue if a provider CLI is available and mark the Notchlet issue link `sync pending`.
- **Neither** — do repository work only and report that tracking is unavailable.

Prefer connected whenever present; never silently fall back to local. Never present local evidence as remote status.

## Entry criteria

- The item meets a Definition of Ready from `notchlet-plan` (measurable acceptance criteria, dependencies, verification).
- The item is resolved from its source: the platform backlog via `notchlet_backlog_get` (connected), or the existing local change (local). Its description, acceptance criteria, children, and linked documents are read.
- The repository is clean or its local changes are understood, and you know the current branch.
- The user has authorized starting development (issue + branch).

## Establish the work

1. Read repository instructions, `git status --short`, and the actual repository/branch (`git rev-parse --show-toplevel`, `git branch --show-current`). Preserve existing user changes.
2. Reuse the authenticated OpenCode/Claude Code MCP connection, or the offline local server when no account is available. Discover live tool names/schemas and never fabricate calls.
3. Connected: resolve the product (`notchlet_workspace_context` / `notchlet_workspace_search`); do not assume it from a previous conversation. Local: skip platform resolution and use the local change.
4. Connected: search for the item by key/title, fetch the match, and read bounded context with `backlog_development_context`. Local: read the local change requirements and checks. Do not dump the full product backlog.
5. If an epic is requested, inspect its children and proceed in dependency order; do not treat one completed child as a completed epic.

## Start development

Perform these steps in order, once authorized:

1. **Create the Git issue.** Connected: use `notchlet_git_issue_create` to create (or reconcile) the issue, or `notchlet_git_issue_link` for an existing issue; verify the returned number/URL. Local: create the provider issue with the provider CLI if available and mark the Notchlet link `sync pending`; otherwise record the issue as deferred. Issue title should reference the backlog key and title.
2. **Create the branch.** Use `<type>/<backlog-key>-<slug>`, for example `feat/APP-42-slack-toggle`, from an up-to-date `main`/`develop`. Choose the type from the change (`feat`, `fix`, `chore`, `docs`, `refactor`).
3. **Checkout the branch locally** and confirm `git branch --show-current`.
4. **Set the item to `IN_PROGRESS`** and confirm the returned state. Connected: update the platform. Local: update the local change/task state and mark remote status `sync pending`. Do not change unrelated items.

## Pre-implementation checklist

- [ ] Item status is `TODO` or `IN_PROGRESS` (never `DONE` or `IN_REVIEW`).
- [ ] All blocking dependencies are resolved or marked non-blocking.
- [ ] Acceptance criteria are measurable and testable.
- [ ] You know the repository's lint, type-check, and test commands.
- [ ] You have located existing implementations before creating new abstractions.
- [ ] The Git issue is linked and the branch is checked out.

## Implement

- State a concise approach and the acceptance checks that will prove it. For small changes, start coding directly.
- Use `notchlet-plan` if the work needs decomposition or missing acceptance criteria.
- Locate existing implementations before adding abstractions. Check package scripts and surrounding code for commands and patterns.
- Implement the complete behavior: authorization, validation, failure handling, edge cases, and caller integration.
- If you edit the item description, pass the latest `expectedDescriptionRevision` and reconcile conflicts. Use comments for progress; do not replace requirements with logs.
- Do not push, open a PR, deploy, or publish without explicit authorization. Linking an issue is not authorization to push.

## Testing standards

Run focused checks suited to the change:

- [ ] Lint passes with no new warnings introduced by the change.
- [ ] Type-check passes (`tsc --noEmit` or equivalent).
- [ ] Unit tests pass for the changed code paths.
- [ ] Integration tests pass if the change touches database, MCP, or external integrations.
- [ ] Pre-existing failures are distinguished from failures caused by the change.

Do not invent results or run auto-fixing lint without accounting for its edits. If a check fails because of your change, fix it before finishing.

## Commit and push

1. Inspect the final diff for unintended files and exposed secrets.
2. Commit with a conventional message that references the backlog key, for example `feat(APP-42): add Slack notification toggle`.
3. Add one `### Agent checkpoint` comment to the backlog item: approach, changed files, exact check outcomes, branch/commit, blockers, next action. Avoid per-tool-call commentary and never include secrets.
4. **Push the branch only when authorized.** Then report the branch and commit SHA. Opening the PR belongs to `notchlet-deliver`.

## Failure handling

- If a check fails because of your change, fix it before marking implementation complete.
- If a check fails for a pre-existing reason, document it as a blocker and do not claim your change caused it.
- If Notchlet is unavailable, continue repository work that does not depend on missing requirements, report the synchronization blocker, and do not claim remote updates succeeded.

## CLI tracking layer (required when available)

The CLI is the executable tracking record for tasks and evidence. Connected: import the item once so tasks/evidence stay linked to the backlog. Local: it is the primary record until sync is possible.

- Import once (connected): `notchlet backlog import <key-or-title>`.
- Author or refine checks/tasks: `notchlet change update|add|apply ...`.
- `notchlet task start|complete <task> --change <change>`.
- `notchlet verify <change> --record` and `notchlet evidence list --change <change>`.
- `notchlet checkpoint --change <change> --name <name>` at meaningful milestones.
- Inspect with `notchlet status <change>` / `notchlet instructions <change> --action implement`.

Never hand-edit `tasks.json`. Never include credentials in evidence. Local evidence is not a remote update. Full command reference: `notchlet-super`.

## Definition of done before deliver

- [ ] All acceptance criteria are met with evidence.
- [ ] All focused checks pass.
- [ ] The diff is reviewed for secrets and unintended files.
- [ ] The branch is committed, and pushed if the user authorized it.
- [ ] The checkpoint comment is recorded.
- [ ] CLI tracking is recorded (connected) or reported as `sync pending` (local).
- [ ] No unresolved blockers remain.

## When NOT to use

- To plan tasks — use `notchlet-plan`.
- To review or deliver completed work — use `notchlet-review` or `notchlet-deliver`.
- To set up the MCP connection — use `notchlet-mcp`.

## Related skills

- `notchlet-plan` — establishes the Definition of Ready
- `notchlet-review` — reviews the PR against the backlog description
- `notchlet-deliver` — opens the PR, runs review, merges, and records evidence
- `notchlet-super` — orchestrates the full cycle and holds the CLI command reference
