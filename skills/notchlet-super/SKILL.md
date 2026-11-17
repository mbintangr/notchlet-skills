---
name: notchlet-super
description: Use when you want to run the complete Notchlet-driven development cycle end-to-end. This meta-skill actively guides you through setup, discovery, planning, implementation, review, and delivery — asking for your authorization at each gate before proceeding to the next phase.
---

# Notchlet Super — Full-Cycle Development Orchestrator

Use this skill when you want to take work from idea to live using Notchlet-driven development. It orchestrates all six skills (`notchlet-mcp`, `notchlet-platform`, `notchlet-plan`, `notchlet-implement`, `notchlet-review`, `notchlet-deliver`) and drives you through each phase interactively, asking for confirmation before moving forward.

## How It Works

| Phase | Skill | Agent Action | Your Authorization |
|-------|-------|--------------|-------------------|
| 0. Init | `notchlet-init` | Generate AGENTS.md and PROMPT.md context files | Confirm workspace setup |
| 1. Setup | `notchlet-mcp` | Configure `opencode.jsonc`, verify MCP connection | Confirm MCP is reachable |
| 2. Discover | `notchlet-platform` | Resolve product + backlog item, read context | Confirm correct item |
| 3. Plan | `notchlet-plan` | Break into tasks, Definition of Ready | Confirm plan is correct |
| 4. Implement | `notchlet-implement` | Write code, run checks, commit locally | Authorize push/PR |
| 5. Review | `notchlet-review` | Check PR against acceptance criteria | Approve or request changes |
| 6. Deliver | `notchlet-deliver` | Record evidence, set `DONE`, monitor | Confirm delivery |

The agent will:

1. **Init** — generate AGENTS.md and PROMPT.md context files (via `notchlet-init`).
2. **Setup** — configure `opencode.jsonc` MCP remote and verify `https://pg.mbintangr.com/mcp` is reachable (via `notchlet-mcp`).
3. **Discover** — resolve the product and backlog item you want to work on.
4. **Plan** — break the work into tasks with a Definition of Ready.
5. **Implement** — write code, run checks, commit (but not push/PR without your authorization).
6. **Review** — review the PR/MR against acceptance criteria (approve or request changes).
7. **Deliver** — record evidence, merge, set status to `DONE`, monitor.

At every phase boundary, the agent pauses and asks you to confirm before continuing. You can stop at any point.

---

## Phase 0: Init (`notchlet-init`)

**Agent asks:** *"Shall I initialize this workspace for Notchlet-driven development — generate AGENTS.md and PROMPT.md context files?"*

If not already present, the agent creates:

- **AGENTS.md** — agent operating instructions (project overview, commands, conventions, testing, Notchlet workflow)
- **PROMPT.md** — project context (tech stack, project structure, key files, conventions)

**Proceed when:** Context files are created (or already present and confirmed). MCP setup is handled in Phase 1.

---

## Phase 1: Setup (`notchlet-mcp`)

**Agent asks:** *"Shall I verify that OpenCode is connected to the Notchlet MCP backend at https://pg.mbintangr.com/mcp?"*

If not configured, the agent guides you to edit `opencode.jsonc`:
```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "notchlet": {
      "type": "remote",
      "url": "https://pg.mbintangr.com/mcp",
      "oauth": false,
      "headers": { "Authorization": "Bearer your-mcp-token" }
    }
  }
}
```

Then restart OpenCode and verify with `curl https://pg.mbintangr.com/mcp`.

**Proceed when:** MCP connection is verified. Otherwise stop here.

---

## Phase 2: Discover (`notchlet-platform`)

**Agent asks:** *"Which Notchlet backlog item should we work on? Please provide the item ID, title, or key."*

The agent:
- Resolves the product via `notchlet_workspace_context`.
- Fetches the item via `notchlet_backlog_get` by ID or searches by title/key.
- If keys are duplicated, asks you to distinguish by title, product, parent, and numeric ID.
- Reads the item's description, acceptance criteria, children, and linked documents.
- Reads bounded context via `backlog_development_context`.

**Agent reports:** The resolved item, its status, acceptance criteria, and any existing children.

**Proceed when:** You confirm this is the correct item to work on.

---

## Phase 3: Plan (`notchlet-plan`)

**Agent asks:** *"Shall I break this item into tasks with acceptance criteria and dependencies?"*

The agent:
- Checks for duplicate existing items to avoid creating redundant work.
- Reads relevant BRD/PRD/SRD/CUSTOM documents and linked tickets.
- Identifies deliverables, exclusions, dependencies, and acceptance criteria.
- Decomposes into epics/stories/tasks proportionately.
- Creates items sequentially, capturing `parentId` for each.
- Verifies the hierarchy in Notchlet.
- Applies the Definition of Ready checklist:
  - [ ] Acceptance criteria are measurable
  - [ ] Dependencies listed or "none"
  - [ ] Scope is specific
  - [ ] Verification criteria defined
  - [ ] No unresolved blocking dependency

**Agent reports:** The planned tasks, their IDs, and that Definition of Ready is met.

**Proceed when:** You confirm the plan looks correct and want to start implementation.

---

## Phase 4: Implement (`notchlet-implement`)

**Agent asks:** *"Shall I start implementing? I will write code and run checks, but I will NOT push, create a PR, or deploy without your explicit authorization."*

The agent:
- Sets the item to `IN_PROGRESS` (with `expectedDescriptionRevision` if supported).
- Creates a feature branch.
- Implements the behavior including authz, validation, failure handling.
- Runs lint, type-check, unit tests, integration tests.
- Inspects diff for secrets and unintended files.
- Adds a checkpoint comment on the backlog item.
- Commits locally.

**Agent pauses and asks:** *"Implementation is complete and checks pass. Shall I push the branch and create a PR/MR?"*

**Proceed when:** You explicitly authorize push and PR creation. Otherwise the agent stops at commit.

---

## Phase 5: Review (`notchlet-review`)

**Agent asks:** *"Shall I review the PR/MR against the acceptance criteria?"*

The agent runs the review checklist:
- [ ] Every acceptance criterion addressed
- [ ] No scope creep
- [ ] Edge cases handled
- [ ] No secrets in diff
- [ ] No unintended files
- [ ] Existing code reused, no unnecessary abstractions
- [ ] Lint, type-check, tests pass

**Verdict options:**
- **Approve:** Sets item to `IN_REVIEW`, notifies you delivery can proceed.
- **Request changes:** Lists specific fixes needed on the PR/MR and backlog item. Returns to implement phase until resolved.

**Proceed when:** Review is approved (or you waive review per team policy).

---

## Phase 6: Deliver (`notchlet-deliver`)

**Agent asks:** *"The PR is merged and review is approved. Shall I finalize delivery — record evidence and set the item to DONE?"*

The agent:
- Runs the Definition of Done checklist:
  - [ ] All acceptance criteria met with evidence
  - [ ] All checks pass
  - [ ] Review approved
  - [ ] PR merged / implementation committed
  - [ ] Parent not completed while children open
  - [ ] No unresolved blockers
- Adds a `### Development evidence` comment to the backlog item.
- Updates the item status to `DONE` and confirms the returned state.
- Does NOT auto-close linked GitHub issues (`RESOLVED` ≠ `CLOSED`).
- Watches for regressions post-delivery.

**Post-delivery:** The agent watches CI/build for a reasonable period. If a regression appears, it reports it and offers to create a follow-up backlog item via `notchlet-plan`.

---

## Entry Criteria (Before Starting)

- [ ] You have an OpenCode installation with Notchlet skills available.
- [ ] Workspace is initialized via `notchlet-init` (AGENTS.md, PROMPT.md).
- [ ] You have a Notchlet MCP bearer token for `https://pg.mbintangr.com/mcp`.
- [ ] You know which backlog item you want to work on (or are ready to search for one).

---

## Example Conversation Flow

```
You: "Use notchlet-super to implement the Slack notification toggle for APP-42."

Agent: Let me start by verifying the MCP connection.
[Checks opencode.jsonc, confirms https://pg.mbintangr.com/mcp is reachable]
✓ MCP connection verified.

Agent: Resolving story APP-42 in product product-guy...
[Fetches item via notchlet_backlog_get, reads acceptance criteria]
Found: "Add Slack notification toggle" — status TODO
Acceptance criteria: user can enable/disable, default disabled, persisted, audit recorded.

Agent: Shall I break this into tasks and verify Definition of Ready? (y/n)
You: yes

Agent: [Creates 4 tasks, verifies hierarchy, Definition of Ready met]
✓ Plan complete. Tasks: T1 migration, T2 endpoint, T3 tests, T4 frontend.

Agent: Shall I start implementation? I'll write code and run checks, but won't push/PR without your authorization. (y/n)
You: yes

Agent: [Sets IN_PROGRESS, creates branch, implements, runs tests, commits]
✓ Implementation complete. Commit a1b2c3d on feature/slack-toggle.

Agent: Implementation done and checks pass. Shall I push and create a PR? (y/n)
You: yes

Agent: [Pushes, opens PR #157]
✓ PR #157 opened against main.

Agent: Shall I review the PR against acceptance criteria? (y/n)
You: yes

Agent: [Runs review checklist]
✓ Review approved. All criteria met.

Agent: PR is merged and review approved. Shall I finalize delivery — record evidence and set APP-42 to DONE? (y/n)
You: yes

Agent: [Runs Definition of Done, adds evidence comment, sets DONE]
✓ APP-42 is DONE. Evidence recorded. Slack notification toggle is live.
```

---

## When to Use

- You have a Notchlet backlog item and want to take it from idea to live without manually orchestrating each phase.
- You want the AI agent to drive the cycle but retain human authorization gates at push/PR/delivery.
- You want full evidence and status synchronization in Notchlet at every step.

## When NOT to Use

- You only need a single phase (e.g., just planning or just implementing) — use the individual skill instead.
- You don't have MCP access to `https://pg.mbintangr.com/mcp` — set up `notchlet-mcp` first.
- You don't want the agent to ask for authorization at each phase boundary — use individual skills directly.

---

## Skills Orchestrated

- `notchlet-init` — workspace bootstrap: context files + MCP config
- `notchlet-mcp` — setup and connection
- `notchlet-platform` — discovery and item resolution
- `notchlet-plan` — task decomposition and Definition of Ready
- `notchlet-implement` — coding, checks, commit
- `notchlet-review` — PR review and approval gate
- `notchlet-deliver` — evidence, status sync, post-delivery monitoring
