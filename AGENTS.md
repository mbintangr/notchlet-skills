# Agent Instructions — Notchlet-Driven Development

## Core Principle
Notchlet is the **source of product requirements and work status**. The repository is the **source of implementation truth**. Never treat a ticket status or backlog description as a substitute for actual code changes, and never treat code changes as a substitute for updating Notchlet status.

## The Notchlet Development Cycle (Route From Current State)

| Phase | Skill | Purpose | Agent Action | Authorization Required |
|-------|-------|---------|--------------|----------------------|
| 0. Init | `notchlet-init` | Optional workspace context | Generate AGENTS.md/PROMPT.md only when requested | Confirm before writing files |
| 1. Setup | `notchlet-mcp` | MCP connectivity | Reuse a valid connection; configure only when needed | Confirm before editing config |
| 2. Discover | `notchlet-platform` | Product + item resolution | `notchlet_workspace_context`, `notchlet_workspace_search`, `notchlet_backlog_get`, `backlog_development_context` | Confirm correct item before proceeding |
| 3. Plan | `notchlet-plan` | Task decomposition | `notchlet_backlog_create`, `notchlet_backlog_update` with Definition of Ready | Confirm plan before implementing |
| 4. Implement | `notchlet-implement` | Code changes | Write code, run checks, commit locally | Authorize before push/PR |
| 5. Review | `notchlet-review` | Quality gate | Review PR/MR against acceptance criteria | Approve or request changes |
| 6. Deliver | `notchlet-deliver` | Evidence + status | `backlog_comment_add`, status update to `DONE`, sync Git issues | Confirm before marking DONE |

## MCP Tool Inventory (Use These, Never Fabricate)

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
- Verify through authenticated MCP client initialization and advertised `tools/list`, or use `notchlet doctor` for the authenticated CLI/workspace binding. Never probe MCP with unauthenticated HTTP requests; use only supported client setup and authenticated diagnostics. Keep MCP credentials out of project files and prompts.
- If MCP is unreachable, continue repository work that does not depend on missing requirements; report the blocker; do not claim remote updates succeeded.

### Phase 2: Discover (`notchlet-platform`)
- Resolve product via `notchlet_workspace_context` or `notchlet_workspace_search`. Never assume product from previous conversation.
- Search for the backlog item by key/title, then read the confirmed match with `notchlet_backlog_get`; use numeric IDs for exact calls and only as a fallback. If keys are duplicated, distinguish by title, product, parent, and numeric ID. Ask user only if still ambiguous.
- Read item description, acceptance criteria, children, assignments, linked documents before acting.
- Use `backlog_development_context` for bounded context. Read specific sections only; do not dump full backlog.
- Read linked BRD/PRD/SRD/CUSTOM documents via `notchlet_document_list` + `notchlet_document_get`.

### Phase 3: Plan (`notchlet-plan`)
- Check for duplicate existing items before creating. Prefer refining existing item when it covers the same work.
- Create epic → stories → tasks sequentially, capturing each returned numeric ID for `parentId`.
- Apply Definition of Ready before implementation:
  - Acceptance criteria are measurable (given/when/then or equivalent)
  - Dependencies listed with existing item titles and numeric IDs, or marked "none"
  - Scope is specific (capability, route, or component named)
  - Verification criteria defined (what test/evidence proves completion)
  - No unresolved blocking dependency remains
- Use description template: Outcome / Scope / Acceptance criteria / Dependencies / Verification
- Keep planning lightweight for small changes; no OpenSpec artifact, proposal/design/spec hierarchy, or new change-management system required.

### Phase 4: Implement (`notchlet-implement`)
- Item must have Definition of Ready met and be in TODO or IN_PROGRESS status.
- Set item to IN_PROGRESS when implementation starts.
- Create feature branch; do not commit directly to main/develop unless authorized.
- Implement complete behavior including authorization, validation, failure handling, caller integration.
- Run focused checks: lint, type-check (`tsc --noEmit` or equivalent), unit tests, integration tests if touching database/MCP/external.
- Inspect final diff for unintended files and exposed secrets.
- Add `### Agent checkpoint` comment on backlog item with approach, changed files, exact check outcomes, actual branch/commit, blockers, next action.
- Commit locally. Do NOT push, create PR, or deploy without explicit user authorization.
- If Notchlet is unavailable, continue repository work; report blocker; do not claim remote updates succeeded.

### Phase 5: Review (`notchlet-review`)
- Implementation complete and committed on feature branch.
- PR/MR open against target branch.
- Review checklist:
  - Every acceptance criterion addressed
  - No scope creep
  - Edge cases and failure behavior handled
  - No exposed secrets or unintended files in diff
  - Existing abstractions reused; no unnecessary new abstractions
  - Authorization, validation, caller integration present
  - Focused checks passed (actual output, not invented)
  - Pre-existing failures distinguished from change-caused failures
- Approve: set item to IN_REVIEW, notify delivery can proceed.
- Request changes: add specific comment on PR/MR and backlog item; return to implement until resolved.

### Phase 6: Deliver (`notchlet-deliver`)
- Definition of Done:
  - All acceptance criteria met with actual evidence (test output, diff, command results)
  - All focused checks pass
  - Code review approved (or not required per team policy)
  - PR/MR merged (branch-linked) or implementation committed (issue-only)
  - No unresolved blockers
  - Parent not marked DONE while required children remain open
- Record `### Development evidence` comment with scope, changed components, verification, acceptance criteria met, Git evidence, blockers, resulting status.
- Update item status to DONE and confirm returned state.
- Do NOT auto-close linked GitHub issues (RESOLVED ≠ CLOSED).
- Watch for regressions post-delivery; if regression appears, report and create follow-up fix item via `notchlet-plan`.

## Safety Rules (Non-Negotiable)

- [ ] Never commit, push, create PR, branch, publish, or deploy without explicit user authorization.
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
- ❌ Pushing/PR-ing without user authorization
- ❌ Marking DONE without review approval
- ❌ Fabricating MCP tool calls or tool schemas
- ❌ Dumping the full product backlog instead of targeted reads
- ❌ Treating stored Git metadata as live verification
- ❌ Replacing user-authored requirements with progress logs in descriptions

## When Notchlet MCP Is Unavailable

- Continue repository work that does not depend on missing requirements (coding, testing, local verification).
- Record intended changes locally so they can be synced when platform returns.
- Report the synchronization blocker explicitly.
- Do not claim remote updates succeeded.
- Do not silently proceed with status changes that cannot be verified on the platform.

## When to Use Each Skill

- `notchlet-init` — optional, user-requested context-file setup (this file)
- `notchlet-mcp` — configure/verify MCP connection
- `notchlet-platform` — discover products, resolve backlog items
- `notchlet-plan` — create/refine epics, stories, tasks with Definition of Ready
- `notchlet-implement` — implement backlog items/tickets
- `notchlet-review` — review PR/MR against acceptance criteria
- `notchlet-deliver` — record evidence, update status, sync issues
- `notchlet-super` — orchestrate the full cycle with authorization gates

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
