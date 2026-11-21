---
name: notchlet-plan
description: Use when turning an idea, ticket, or document into Notchlet backlog items. Grills the request until requirements are unambiguous, then creates or refines epics, stories, and tasks with measurable acceptance criteria, dependencies, verification, and a Definition of Ready before implementation begins.
---

# Plan executable Notchlet backlogs

Planning has two jobs: force the ambiguity out first, then persist a backlog that an implementer can execute without guessing. Do not create items until the grilling reaches a Definition of Ready.

## Capability mode (check first)

- **Connected** — authenticated MCP tools are advertised, or `notchlet doctor` reports a workspace binding. The platform is the authority for requirements and status: create and verify the backlog there, then import the item with `notchlet backlog import` and author the executable checks/tasks locally.
- **Local** — no account/platform MCP, but the CLI or offline server is available. Author the plan as a local change (`notchlet change update/add`, `notchlet validate`) and record platform backlog creation as `sync pending`.
- **Neither** — plan in conversation only and report that backlog and tracking are unavailable.

Prefer connected whenever present; never silently fall back to local. Never present a local plan as a platform backlog item.

## Phase A — Grill before planning

Interview the user before writing requirements. Ask in small batches (about 1–3 questions at a time), one decision per question, and adapt based on the answers. Do not dump a questionnaire.

Cover at least:

- **Outcome:** what observable result must exist when this is done?
- **Users and primary path:** who uses it, and what is the happy path?
- **Scope boundaries:** what is explicitly included and explicitly out of scope?
- **Failure modes:** what happens on invalid input, timeout, unauthorized access, partial failure, or empty state?
- **Data and integrations:** does it touch persistence, migrations, external services, permissions, or configuration?
- **Acceptance:** what measurable checks prove each behavior (given/when/then or equivalent)?
- **Dependencies:** what must exist or complete first, with existing item titles and numeric IDs?
- **Verification:** which tests, commands, or evidence demonstrate completion?

Rules:

- Challenge vague answers; "make it better" or "handle errors" is not acceptance criteria.
- Separate stated requirements from assumptions, and ask about genuinely blocking ambiguity only.
- Stop grilling once every Definition of Ready box can be filled. Summarize the resolved requirements back to the user.
- Keep the conversation lightweight for small changes; scale depth to risk, not ceremony.

## Phase B — Discover scope

1. Resolve the product from live workspace context. Search by the requested key/title, read the existing item and bounded context, and avoid duplicates. Prefer refining an existing item that covers the same work.
2. Read relevant BRD/PRD/SRD or CUSTOM documents and linked tickets.
3. Identify deliverables, exclusions, dependencies, and acceptance criteria from the grilling.
4. Before decomposing an epic or story, inspect its current children, reuse/update matching children, and create only missing work.
5. In local mode, skip platform discovery and operate on the existing local change instead.

## Decompose proportionately

- **Epic:** outcome and measurable completion conditions across the initiative.
- **Story:** a coherent capability verifiable independently, with acceptance criteria and dependency titles/IDs.
- **Task:** actionable implementation or verification work with concrete scope, expected artifacts, and a check proving completion.
- Keep planning lightweight. For a small code change, use the existing item description with a concise outcome and acceptance checks; do not require OpenSpec artifacts, a proposal/design/spec hierarchy, or a new spec/change-management system.
- Include frontend/caller changes, data implications, and release work when the behavior requires them.
- Do not infer assignments, due dates, or urgency without requirements.
- Estimate only when useful, and label estimates provisional.

## Definition of Ready

A story or task is ready for `notchlet-implement` only when:

- [ ] Acceptance criteria are written and measurable (given/when/then or equivalent).
- [ ] Dependencies are listed with existing item titles and numeric IDs, or marked "none".
- [ ] Scope is specific: the capability, route, or component is named.
- [ ] Verification criteria are defined (what test or evidence proves completion).
- [ ] No unresolved blocking dependency remains.

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
2. Check for successful writes before retrying errors. Track partial creation so retries do not duplicate items.
3. Read the root item after creation and verify child counts, types, product, parent links, descriptions, and estimates.
4. Report verified totals and a compact hierarchy. Report service anomalies explicitly.

For existing work, fetch the latest description before editing and preserve its requirements, evidence, and user-authored content. Pass `expectedDescriptionRevision` on description updates when the live schema supports it; reread and reconcile conflicts.

**Mode behavior:** connected — persist on the platform (steps 1–4), then import the item to a local change and author checks/tasks there. Local — author and validate the local change; skip platform creation and mark the backlog as `sync pending`.

## CLI tracking layer (required when available)

The CLI holds the executable plan (requirements, checks, scenarios, tasks) linked to the item:

- `notchlet instructions <change> --action plan` for local planning guidance.
- `notchlet change update <change> ...` and `notchlet change add <change> requirement|check|scenario|decision|task ...`, or atomic `notchlet change apply <change> --file <plan.json>`.
- `notchlet validate <change>` for local readiness.

In connected mode, import the confirmed backlog item before authoring. In local mode, the local change is the plan until it can be synced. A backlog item does not prove local work exists, and a local plan does not replace platform requirements. Full command reference: `notchlet-super`.

## Exit criteria

- Every planned item meets the Definition of Ready.
- Connected: the epic → stories → tasks hierarchy is verified on the platform and mirrored in the local change.
- Local: the local change plan is validated, and platform backlog creation is reported as `sync pending`.
- The first executable item is ready to start.

## When NOT to use

- To implement an already-planned item — use `notchlet-implement`.
- To review or deliver completed work — use `notchlet-review` or `notchlet-deliver`.
- To set up the MCP connection — use `notchlet-mcp`.

## Related skills

- `notchlet-platform` — resolve the product and backlog item before planning
- `notchlet-implement` — implement after the Definition of Ready is met
- `notchlet-super` — orchestrates the full cycle and holds the CLI command reference
