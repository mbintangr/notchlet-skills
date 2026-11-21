# Agent Instructions — Notchlet-Driven Development

## Core Principle
Notchlet is the **source of product requirements and work status**. The repository is the **source of implementation truth**. Never treat a ticket status or backlog description as a substitute for actual code changes, and never treat code changes as a substitute for updating Notchlet status.

## The Notchlet Development Cycle (Base Flow)

| Phase | Skill | Purpose | Agent Action | Authorization Required |
|-------|-------|---------|--------------|----------------------|
| 0. Init | `notchlet-init` | Optional workspace context | Generate AGENTS.md/PROMPT.md only when requested | Confirm before writing files |
| 1. Setup | `notchlet-mcp` | MCP connectivity | Reuse a valid connection; configure only when needed | Confirm before editing config |
| 2. Discover | `notchlet-platform` | Product + item resolution | `notchlet_workspace_context`, `notchlet_workspace_search`, `notchlet_backlog_get`, `backlog_development_context` | Confirm correct item before proceeding |
| 3. Plan | `notchlet-plan` | Grill + task decomposition | Interview until requirements are unambiguous, then `notchlet_backlog_create` / `notchlet_backlog_update` with Definition of Ready | Confirm grilled plan before creating backlogs and implementing |
| 4. Implement | `notchlet-implement` | Git issue, branch, code | `notchlet_git_issue_create`/`link`, create `<type>/<key>-<slug>` branch, checkout, develop, run checks, commit, push | Confirm issue + branch; authorize push |
| 5. Deliver | `notchlet-deliver` (review via `notchlet-review`) | PR, review, merge, evidence, status | Open PR, review against the backlog description, fix/commit/push, merge, record evidence, set `DONE`, sync Git issues | Confirm PR, merge, and DONE |

The base flow is Git-centric. The Notchlet CLI (`notchlet change/task/verify/evidence/status/sync/checkpoint`) is the primary local tracking layer whenever available: it adds local detail and evidence while remaining linked to the backlog, but it never replaces the Git flow. When resuming, inspect the existing item, issue, branch, PR, and local change, then continue from the first incomplete phase instead of restarting completed ones.

## Capability Modes

Decide the mode once per session and reuse it unless connectivity changes:

| Mode | Detection | Authority for requirements/status | Primary tracking |
|------|-----------|-----------------------------------|------------------|
| Connected | Authenticated MCP advertises `tools/list`, or `notchlet doctor` reports a workspace binding | Notchlet platform | CLI local change/task/verify/evidence, linked to the backlog |
| Local | No account/platform MCP, but the CLI or offline server is available | Local change model | CLI local change/task/verify/evidence |
| Neither | No MCP and no CLI | Repository only | None; report the blocker |

Rules:

- Prefer connected whenever present. Never silently fall back to local because local always works.
- Phase rules below describe connected mode. In local mode, replace platform backlog/issue/status steps with local change/task/evidence steps and mark them `sync pending`.
- Never present local evidence, local closure, or local status as a remote update.
- When connectivity returns, run `notchlet sync status --change <change>`, resolve divergence, and sync only when authorized.
- In neither mode, do repository-only work and report that tracking is unavailable.

## MCP Tool Inventory (Use These, Never Fabricate)

Names below show the OpenCode-exposed form. The server registers the bare tool names (`workspace_context`, `backlog_get`, `development_instructions_get`, ...) and Claude Code exposes `mcp__notchlet__*`. Call the exact names returned by `tools/list`.

### Workspace Context
- `notchlet_workspace_context` — list accessible organizations, teams, products
- `notchlet_workspace_search` — search products, documents, backlog items, tickets by query

### Backlog Items
- `notchlet_backlog_get` — fetch one backlog item by ID (always read before acting)
- `notchlet_backlog_create` — create epic/story/task/subtask (capture returned numeric ID for parentId)
- `notchlet_backlog_update` — update title/description/status/priority/storyPoints (use expectedDescriptionRevision on description edits)
- `notchlet_backlog_update_status` — change status (TODO, IN_PROGRESS, IN_REVIEW, DONE)

### Development Context
- `backlog_development_context` — bounded context for one item (itemId; optional section: item|ancestors|children|comments|document|repositories; cursor; limit; knownFingerprint). Response capped at 32 KiB serialized bytes; may include truncation metadata and scoped cursors. Follow cursors with same limit; do not assume a truncated page is complete.

### Development Workflow
- `development_capabilities` — negotiated protocol/core versions and request limits
- `development_changes_list` / `development_change_get` — discover and read changes
- `development_change_create` — create a change; `development_snapshot_upload`, `development_snapshots_list`, `development_snapshot_get` — immutable snapshots keyed by expected revision and operation identity
- `development_instructions_get` — server-evaluated instructions tied to a synchronized revision
- `development_checkpoint_create` — synchronized checkpoint (not delivery evidence by itself)
- `development_delivery_create` — explicit, idempotent delivery evaluation; query `development_operation_get` after an uncertain response

### Comments & Activities
- `backlog_comments_list` — read comments on a backlog item
- `backlog_comment_add` — record checkpoint comments (approach, changed files, check outcomes, blockers, next action)
- `backlog_activities_list` — read activity history for a backlog item

### Git Issues
- `notchlet_git_issue_create` — create/reconcile GitHub/GitLab issue for a backlog item
- `notchlet_git_issue_link` — link backlog item to existing issue number or URL
- `notchlet_git_issue_sync` — synchronize backlog item issue to remote (safe to retry)
- `notchlet_git_issues_list` — list remote issues connected to a product

### Documents
- `notchlet_document_generate_brd` / `prd` / `srd` — start document generation (BRD must exist before PRD; PRD before SRD)
- `notchlet_document_get` — read a document by ID
- `notchlet_document_list` — list product documents
- `notchlet_document_save` — create/revise a document draft (never approves or commits)

### Tickets
- `notchlet_tickets_convert_to_backlog` — convert ticket to backlog item
- `notchlet_tickets_create` — create a ticket in a product
- `notchlet_tickets_get` — read a ticket
- `notchlet_tickets_list` — list tickets in a product
- `notchlet_tickets_update_status` — update ticket status (OPEN, ON_GOING, RESOLVED, CLOSED)

## Phase-by-Phase Rules

### Phase 0: Optional Init (`notchlet-init`)
- Do not generate AGENTS.md or PROMPT.md as an automatic prerequisite for using Notchlet.
- Only create or revise context files when the user requests workspace context setup.
- Preview changes, never overwrite existing files without explicit approval, and preserve manually edited sections.

### Phase 1: Setup (`notchlet-mcp`)
- Reuse a valid user-managed MCP connection. Do not reconfigure it or ask the user to repeat setup when authenticated tools are already available.
- Without an account, use the offline local server: `notchlet init --local --client opencode` (or `claude`), or `notchlet mcp configure --client opencode --local`. Refresh installed guidance without touching MCP config with `notchlet client update <agent> --apply`.
- Verify through authenticated MCP client initialization and advertised `tools/list`, or use `notchlet doctor` for the authenticated CLI/workspace binding. Never probe MCP with unauthenticated HTTP requests; use only supported client setup and authenticated diagnostics. Keep MCP credentials out of project files and prompts.
- If MCP is unreachable, continue repository work that does not depend on missing requirements; report the blocker; do not claim remote updates succeeded.

### Phase 2: Discover (`notchlet-platform`)
- Resolve product via `notchlet_workspace_context` or `notchlet_workspace_search`. Never assume product from previous conversation.
- Search for the backlog item by key/title, then read the confirmed match with `notchlet_backlog_get`; use numeric IDs for exact calls and only as a fallback. If keys are duplicated, distinguish by title, product, parent, and numeric ID. Ask user only if still ambiguous.
- Read item description, acceptance criteria, children, assignments, linked documents before acting.
- Use `backlog_development_context` for bounded context. Read specific sections only; do not dump full backlog.
- Read linked BRD/PRD/SRD/CUSTOM documents via `notchlet_document_list` + `notchlet_document_get`.

### Phase 3: Plan (`notchlet-plan`)
- Grill the request before writing anything: ask, in small batches, about the observable outcome, users and primary path, in/out of scope, failure modes, data/integration/permission impact, acceptance checks, dependencies, and verification. Stop when the Definition of Ready can be met, and summarize the resolved requirements back to the user.
- Check for duplicate existing items before creating. Prefer refining existing item when it covers the same work.
- Create epic → stories → tasks sequentially, capturing each returned numeric ID for `parentId`.
- Apply Definition of Ready before implementation:
  - Acceptance criteria are measurable (given/when/then or equivalent)
  - Dependencies listed with existing item titles and numeric IDs, or marked "none"
  - Scope is specific (capability, route, or component named)
  - Verification criteria defined (what test/evidence proves completion)
  - No unresolved blocking dependency remains
- Use description template: Outcome / Scope / Acceptance criteria / Dependencies / Verification
- Local mode: author the plan as a local change (`notchlet change update/add`, `notchlet validate`) and mark platform backlog creation `sync pending`.
- Keep planning lightweight for small changes; no OpenSpec artifact, proposal/design/spec hierarchy, or new change-management system required.

### Phase 4: Implement (`notchlet-implement`)
- Item must have Definition of Ready met and be in TODO or IN_PROGRESS status.
- With authorization to start development, create or link the Git issue (`notchlet_git_issue_create` / `notchlet_git_issue_link`) and verify the returned issue number/URL.
- Create the feature branch as `<type>/<backlog-key>-<slug>` from an up-to-date main/develop, then checkout locally. Do not commit directly to main/develop unless authorized.
- Set item to IN_PROGRESS when implementation starts.
- Implement complete behavior including authorization, validation, failure handling, caller integration.
- Run focused checks: lint, type-check (`tsc --noEmit` or equivalent), unit tests, integration tests if touching database/MCP/external.
- Inspect final diff for unintended files and exposed secrets.
- Commit with a conventional message referencing the backlog key. Push only when authorized; opening the PR belongs to delivery.
- Add `### Agent checkpoint` comment on backlog item with approach, changed files, exact check outcomes, actual branch/commit, blockers, next action.
- CLI tracking (required when available): import with `notchlet backlog import <key-or-title>`, author with `notchlet change update` / `change add` / `change apply` (never hand-edit `tasks.json`), declare runnable `--command` checks, then `notchlet verify <change> --record`, `notchlet evidence list --change <change>`, and `notchlet status <change>`.
- Local mode: use the local change for tasks/verify/evidence; create the provider issue if available and mark the Notchlet issue link `sync pending`.
- If Notchlet is unavailable, continue repository work; report blocker; do not claim remote updates succeeded.

### Phase 5: Deliver (`notchlet-deliver`, review via `notchlet-review`)
- Implementation is pushed on the feature branch.
- Open the PR/MR into the target branch with the backlog key in the title/body and the Git issue linked; verify the returned PR number/URL. Set the item to IN_REVIEW.
- Review against the backlog description and acceptance criteria:
  - Every acceptance criterion addressed
  - No scope creep
  - Edge cases and failure behavior handled
  - No exposed secrets or unintended files in diff
  - Existing abstractions reused; no unnecessary new abstractions
  - Authorization, validation, caller integration present
  - Focused checks passed (actual output, not invented)
  - Pre-existing failures distinguished from change-caused failures
- Request changes: add specific comments on the PR and backlog item; return to implement, fix, commit, push, and re-review the changed scope.
- Approve, then merge the PR with authorization and verify it is actually merged (closed-unmerged is not merge).
- Definition of Done:
  - All acceptance criteria met with actual evidence (test output, diff, command results)
  - All focused checks pass
  - Code review approved (or not required per team policy)
  - PR/MR merged (branch-linked) or implementation committed (issue-only)
  - No unresolved blockers
  - Parent not marked DONE while required children remain open
- Record `### Development evidence` comment with scope, changed components, verification, acceptance criteria met, Git evidence, blockers, resulting status.
- CLI tracking (required when available): record local evidence with `notchlet verify <change> --record`, inspect `notchlet evidence list --change <change>`, and use `notchlet change close` / `notchlet sync push` only when authorized. Local closure and remote delivery are separate outcomes; do not claim a remote backlog update occurred when only local evidence was recorded.
- Local mode: deliver through Git where a remote exists, record local evidence, close locally with `notchlet change close`, and mark remote `DONE` and issue sync `sync pending` until connectivity returns.
- Update item status to DONE and confirm returned state.
- Do NOT auto-close linked GitHub issues (RESOLVED ≠ CLOSED). Sync the issue only when authorized.
- Watch for regressions post-delivery; if regression appears, report and create follow-up fix item via `notchlet-plan`.

## Safety Rules (Non-Negotiable)

- [ ] Never push, open or merge a PR, create a remote issue, publish, deploy, or change remote status without explicit user authorization. Local commits stay scoped to the authorized item.
- [ ] Prefer connected mode whenever platform MCP is available; never silently default to local-only.
- [ ] Never present local evidence, local closure, or local status as a remote update.
- [ ] Never write secrets, keys, credentials, or personal data into context files, comments, or code.
- [ ] Never claim an MCP tool succeeded if the call returned an error or MCP is unreachable.
- [ ] Never overwrite another user's edits without reconciling via `expectedDescriptionRevision`.
- [ ] Never treat a local commit or closed-unmerged PR as proof of feature completion; merge verification required for branch-linked work.
- [ ] Never mark parent DONE while required children remain open.
- [ ] Never invent results, test output, or check outcomes.
- [ ] Never treat `RESOLVED` ticket status as `CLOSED`.

## Anti-Patterns to Avoid

- ❌ Updating backlog status without implementing the behavior
- ❌ Implementing without reading acceptance criteria first
- ❌ Creating duplicate backlog items instead of refining existing ones
- ❌ Skipping Definition of Ready before implementation
- ❌ Pushing, PR-ing, or merging without user authorization
- ❌ Marking DONE without review approval
- ❌ Creating a duplicate Git issue or branch when the item already has one
- ❌ Reviewing a PR without reading the backlog description and acceptance criteria
- ❌ Silently falling back to local mode when platform MCP is available
- ❌ Fabricating MCP tool calls or tool schemas
- ❌ Dumping the full product backlog instead of targeted reads
- ❌ Treating stored Git metadata as live verification
- ❌ Replacing user-authored requirements with progress logs in descriptions

## Local Mode (No Account)

- Continue repository work that does not depend on missing requirements (coding, testing, local verification).
- Use the offline local server when configured (`notchlet init --local --client opencode`) or the CLI directly: `notchlet change update` / `change add`, `notchlet task start|complete`, `notchlet verify <change> --record`, and `notchlet evidence list --change <change>`.
- Author the local change as the working requirements/plan source and record intended platform actions (backlog creation, issue linking, status) as `sync pending`.
- Do not claim remote updates succeeded.
- When connectivity returns, run `notchlet sync status --change <change>`, resolve divergence, and sync only when authorized.
- Do not silently proceed with status changes that cannot be verified on the platform.

## When to Use Each Skill

- `notchlet-init` — optional, user-requested context-file setup (this file)
- `notchlet-mcp` — configure/verify MCP connection
- `notchlet-platform` — discover products, resolve backlog items
- `notchlet-plan` — grill requirements, then create/refine epics, stories, tasks with Definition of Ready
- `notchlet-implement` — create/link the Git issue, create and checkout the branch, implement, check, commit, push
- `notchlet-review` — review the PR against the backlog description and acceptance criteria
- `notchlet-deliver` — open the PR, run review, drive fixes, merge, record evidence, set DONE
- `notchlet-super` — orchestrate the full cycle with authorization gates and hold the CLI command reference

## Repository Context

- Name: notchlet-skills
- Type: OpenCode skills repository
- Description: OpenCode skills for Notchlet-driven development, formatted for skills.sh
- Root: .

## Commands
- Install skills: npx skills add mbintangr/notchlet-skills
- Install single skill: npx skills add mbintangr/notchlet-skills --skill <skill-name> -a opencode
- Use without installing: npx skills use mbintangr/notchlet-skills@<skill-name> --agent opencode

## Conventions
- Each skill is a directory under skills/ containing a SKILL.md file
- SKILL.md uses YAML frontmatter (name, description) followed by markdown content
- Follows the skills.sh skill format: https://github.com/vercel-labs/skills/blob/main/docs/creating-skills.md
- Conventional commits (feat:, fix:, chore:, docs:)
- Do not commit secrets or keys; use environment variables

## Testing
- Framework: none
- Coverage threshold: none
- Run: n/a

## Deployment
- Build output: none
- Deploy: n/a (published via skills.sh / GitHub)
