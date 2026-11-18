---
name: notchlet-super
description: Route a Notchlet development request from current workspace state; resume existing work without restarting setup or planning.
---

# Notchlet Super — state-aware router

Use this skill to route a request to the smallest relevant Notchlet action. Read current state first; do not replay a generic six-phase setup for every request. Ask only for material decisions or actions that need user authorization.

## Route from the user's intent

- **Explore / investigate:** inspect source, tests, documentation, or MCP context read-only. Do not initialize a workspace, create a change, edit code, or write backlog state unless the user asks.
- **Plan only:** use `notchlet-plan`. Do not implement or deliver. A local change plan can be created/refined without starting tasks or writing application code.
- **Continue / resume:** reuse the identified workspace, product, repository, backlog item, and active change while they remain valid. Run `notchlet status <change-id>` and `notchlet instructions <change-id> --action resume`; continue only eligible tasks. Do not recreate the plan or restart setup when current state is valid.
- **Revise requirements or design:** use `notchlet-plan` and `notchlet instructions <change-id> --action revise`; preserve stable IDs and existing progress. Revalidate before implementation.
- **Implement / verify:** use `notchlet-implement`, `notchlet instructions <change-id> --action implement|verify`, and the repository's actual test commands. Mark a task complete only after its verification evidence exists.
- **Review:** use `notchlet-review`; distinguish a self-recorded check from independent human review.
- **Deliver:** use `notchlet-deliver`. A branch-linked item requires the exact linked PR/MR to be merged and approved before a `DONE` status request. Delivery only evaluates existing provider state; it does not merge or deploy.

## Reuse current workspace and connection

1. Use the nearest initialized `.notchlet` workspace or the user's explicit `--workspace <path>`.
2. If there is no initialized workspace, local reads are side-effect free. Run `notchlet init` only when setup is actually required and the user approves it; do not assume a separate bootstrap command exists.
3. For connected work, use `notchlet doctor` to test authenticated capabilities and identity. Do not probe `/mcp` with an unauthenticated `GET`; Notchlet MCP uses authenticated Streamable HTTP `POST` requests.
4. Do not transfer numeric product IDs between servers. Rebind only after confirming the intended server/product/repository identity.
5. Fetch bounded context for the selected backlog item; never dump the entire product to find one item.
6. If MCP credentials or connection settings are genuinely missing, use `notchlet-mcp` to configure and verify them; otherwise skip setup and reuse the working binding.
7. If the requested backlog item or product is not yet identified, use `notchlet-platform`; if the identity is already confirmed, reuse it instead of searching again.

## State-aware MCP tool calls

The local Notchlet MCP registry exposes these concrete tool names and arguments; use the connected server's `tools/list` schemas as authoritative. Some hosts wrap tool names with a `notchlet_` prefix; call only the exact names actually advertised, and do not assume the new development tools are deployed on every server:

- `backlog_development_context({ itemId, section?, cursor?, knownFingerprint?, limit? })` for one bounded item context.
- `development_changes_list({ productId?, limit?, cursor? })` and `development_change_get({ changeId })` to discover/read changes.
- `development_instructions_get({ changeId, action, snapshotRevision?, context? })` for server-evaluated instructions tied to a synchronized revision.
- `development_delivery_create({ changeId, expectedRevision, operationId, payloadHash, snapshotRevision, target, backlogStatus?, repositoryRef?, summary? })` for an explicit, idempotent delivery evaluation. `backlogStatus` additionally requires `backlog:write`; query the operation after an uncertain response and use a new operation ID only for a new attempt.

The REST API and MCP tools call the same `DevelopmentService`; preserve their expected revisions, operation IDs, payload hashes, scope checks, and error recovery semantics. Do not infer that remote snapshot state represents unsynchronized local edits.

## Authorization and handoff

- Preserve explicit authorization boundaries. Never push, open/merge a PR, deploy, publish skills, or change remote status merely because a task is complete.
- Use `notchlet sync status|push|pull --change <id>` for explicit snapshot synchronization. Divergence requires review; use `notchlet sync resolve --change <id> --keep-local` only after reviewing the local artifacts, then retry conditionally.
- `notchlet checkpoint --change <id> --name <name>` records a synchronized checkpoint; it is not delivery evidence by itself.
- Local verification, local closure, remote delivery, merge, and deployment are separate outcomes. Never describe merge as deployment or a requested status as successful until the service response confirms it.
