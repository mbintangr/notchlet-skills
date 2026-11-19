---
name: notchlet-plan
description: Use when creating or refining Notchlet epics, stories, and tasks from product documents, tickets, or existing workflows, with acceptance criteria, dependencies, estimates, and verified hierarchy. Produces a definition of ready before implementation begins.
---

# Plan executable Notchlet backlogs

Reuse authenticated OpenCode/Claude Code MCP tools and discover their live names/schemas before calls. Native `backlog_development_context`, `backlog_comments_list`, `backlog_comment_add`, and `backlog_activities_list` may be available; use targeted-read/comment fallbacks when not advertised. Context is bounded to 32 KiB serialized bytes, may include truncation metadata and scoped cursors, and should be expanded only as needed. Prefer a backlog key/title for identification and keep numeric IDs for exact references and parent relationships.

## Entry criteria

- Product is resolved via `notchlet_workspace_context` or `notchlet_workspace_search`.
- Source documents (BRD/PRD/SRD/CUSTOM) or linked tickets are available and read.
- You know what outcome is requested; if not, ask before planning.

## Discover scope

1. Resolve the product from live workspace context. Search by the requested key/title, read the existing item and bounded context, and avoid duplicates. Prefer refining an existing item when it covers the same requested work; use its numeric ID only after verifying product, title, key, and parent.
2. Read relevant BRD/PRD/SRD or CUSTOM documents and linked tickets. For migrations, inspect actual source workflows and code callers before claiming detailed coverage.
3. Identify deliverables, exclusions, dependencies, and acceptance criteria. Never substitute generic migration titles for inspected behavior.
4. Before decomposing an epic or story, inspect its current children, reuse/update matching existing children, and create only missing work. Do not duplicate already represented tasks.

## Risks and blockers

- Identify upstream dependencies (other items, teams, or external services) that must complete first.
- Flag items that cannot start until a dependent item reaches `DONE`.
- Mark items with uncertain requirements as needing clarification before implementation.

## Decompose proportionately

- **Epic:** outcome and measurable completion conditions across the initiative.
- **Story:** a coherent user/system capability that can be verified independently. Include acceptance criteria and dependency titles/IDs.
- **Task:** actionable implementation or verification work with concrete scope, expected artifacts, and a check demonstrating completion.
- Keep planning lightweight. For a small code change, use the existing item description with a concise outcome and acceptance checks; do not require OpenSpec artifacts, create a proposal/design/spec-document hierarchy, or introduce a new spec/change-management system. Hosted OpenCode is not a prerequisite.
- Keep an evolving short approach in the description or a comment. Do not replace user-authored requirements with progress logs.
- Avoid overlapping tasks that implement the same runtime/provider/configuration behavior twice. Include frontend/caller changes, data implications, and release work when required by the source behavior.
- Estimate only when useful. Label estimates provisional. Treat story estimates and task estimates as alternative rollups; never add both into one total.
- Do not infer assignments, due dates, or urgency without requirements.

## Definition of ready (before implementation)

A story or task is ready for `notchlet-implement` only when:

- [ ] Acceptance criteria are written and measurable (given/when/then or equivalent).
- [ ] Dependencies are listed with existing item titles and numeric IDs, or marked "none".
- [ ] Scope is specific: capability, route, or component is named.
- [ ] Verification criteria are defined (what test or evidence proves completion).
- [ ] No unresolved blocking dependency remains.

If any readiness condition is unmet, identify the exact gap and clarify or refine the existing item before implementation. A local CLI change/task plan does not replace platform requirements; likewise, a backlog item alone does not prove local work exists. Use `notchlet instructions <change> --action plan` and `notchlet validate <change>` for local change readiness when applicable; do not start tasks or implementation until the requested plan and local preconditions are clear.

Description template:

```markdown
## Outcome
<observable result and source requirement>

## Scope
- <specific capability, route, or component>

## Acceptance criteria
- <given/when/then or measurable check>

## Dependencies
- <existing item title and numeric ID, or none>

## Verification
- <test/evidence required>
```

## Persist and verify

1. Create the epic, then stories, then tasks **sequentially**, capturing each returned numeric ID for `parentId`. Do not guess key sequences or rely on display keys being unique.
2. Check for successful writes before retrying errors. Keep track of partial creation so retries do not duplicate items.
3. Read the root item after creation and verify child counts, types, product, parent links, descriptions, and estimates. If the hierarchy is truncated, fetch children individually.
4. Report verified totals and a compact hierarchy. Report service anomalies explicitly; do not claim keys/parents were repaired using unsupported operations.

For existing work, fetch the latest description before editing and preserve its requirements, evidence, and user-authored content. Pass `expectedDescriptionRevision` on description updates when the live schema supports it; reread and reconcile conflicts. Prefer explicit create operations over batch generation unless the batch payload contract is established.

## Exit criteria

- All planned items have a definition of ready met.
- Hierarchy (epic → stories → tasks) is verified in Notchlet.
- Items are in the correct status (typically `TODO` or `IN_PROGRESS` for the first executable item).

## When NOT to use

- When you only need to implement an already-planned item — use `notchlet-implement` instead.
- When you only need to review or deliver completed work — use `notchlet-review` or `notchlet-deliver` instead.
- When you need to set up the MCP connection first — use `notchlet-mcp` instead.

## Related skills

- `notchlet-platform` — resolve the product and backlog item before planning
- `notchlet-implement` — implement tasks after planning establishes Definition of Ready
- `notchlet-super` — orchestrates the full cycle including planning

## When to use

- Creating or refining epics, stories, and tasks.
- Before implementation begins, to establish a definition of ready.
- When scope, dependencies, or acceptance criteria need clarification.
