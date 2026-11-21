---
name: notchlet-platform
description: Use when discovering the Notchlet platform, resolving products and backlog items, or understanding available MCP operations and limitations before planning, implementing, or delivering work.
---

# Notchlet Platform Discovery

Use this skill first to understand what is available on the connected Notchlet platform before planning, implementing, or delivering work.

## Discover the platform

1. Reuse the authenticated OpenCode or Claude Code MCP connection. Read available operations and limitations from the connected `notchlet_*` host; discover live tool names and schemas and never fabricate calls. If tools are absent, follow `notchlet-mcp` rather than guessing endpoint/config details.
2. Use `notchlet_workspace_context` to list accessible organizations, teams, and products.
3. Use `notchlet_workspace_search` to search products, documents, backlog items, and tickets by query.
4. Resolve a product by ID or name before creating or refining backlog items under it.

Server-side tool names are `workspace_context`, `workspace_search`, `backlog_get`, `backlog_development_context`, and so on. Clients namespace them under the configured server: the examples below use the OpenCode-style `notchlet_*` form, while Claude Code exposes `mcp__notchlet__*`. Always call the exact names returned by `tools/list`.

## Resolve backlog items

1. Prefer the requested backlog key and title to find the existing item with `notchlet_workspace_search`; numeric ID is a fallback. Read the match with `notchlet_backlog_get` and verify key, title, product, and parent before acting.
2. If keys or titles are duplicated, distinguish by product, parent, and numeric ID. Ask only if still ambiguous.
3. Read the item's description, acceptance criteria, children, assignments, and linked documents before acting. Preserve its key/title as agent-facing references and retain numeric IDs for exact tool calls/parent references; do not pass credentials or secrets in queries.
4. Use `backlog_development_context` (`itemId`; optional `section`, `cursor`, `knownFingerprint`, `limit`) for bounded item/ancestor/child/comment/document/repository/Git context. Its response is capped at 32 KiB serialized bytes and may include truncation metadata and scoped cursors; expand only what is needed, follow cursors with the same limit, and do not assume a truncated page is complete.

### Example MCP tool invocations

```json
// Resolve workspace context to find products
{
  "tool": "notchlet_workspace_context"
}

// Fetch a confirmed backlog item by ID after searching its key/title
{
  "tool": "notchlet_backlog_get",
  "arguments": { "id": 42 }
}

// Get bounded development context for an item
{
  "tool": "backlog_development_context",
  "arguments": { "itemId": 42, "limit": 10 }
}
```


## Conflict detection

- Before editing a description, read the latest version and note its `descriptionRevision`.
- If another agent or user has modified the item since you last read it, reconcile conflicts before writing: reread, merge changes, and pass the latest `expectedDescriptionRevision`.
- Never overwrite another user's edits without reconciling first.

## Comments and activities

1. Use `backlog_comments_list` / `backlog_comment_add` to read or record checkpoint comments on a backlog item.
2. Use `backlog_activities_list` to read the activity history for a backlog item.

## Offline / degraded mode

If no authenticated Notchlet MCP host is reachable:

- Continue repository work that does not depend on missing requirements (coding, testing, local verification).
- Use the offline local server when configured (`notchlet init --local --client opencode`): its `notchlet_*` tools manage changes, tasks, verification, and evidence without an account.
- Otherwise use the CLI: `notchlet change new`, `notchlet change update` / `change add`, `notchlet task start|complete`, `notchlet verify <change>`, and `notchlet evidence list --change <change>`.
- Do not claim remote updates succeeded.
- Record your intended changes locally so they can be synced when the platform returns.
- Report the synchronization blocker; do not silently proceed with status changes that cannot be verified on the platform.

## When to use

- Before any planning, implementation, or delivery: resolve the product and item first.
- When you need to understand what MCP tools are available on the connected Notchlet host.
- When you need to search across products, documents, backlog items, or tickets.
- When you need bounded context for one backlog item without dumping the full backlog.
- When you suspect a conflict on an item's description or status.

## When NOT to use

- When the product and backlog item are already resolved and you only need to plan, implement, or deliver — use the individual skill instead.
- When you need to configure the MCP connection itself — use `notchlet-mcp` first.

## Related skills

- `notchlet-mcp` — configure MCP connection before using platform tools
- `notchlet-plan` — plan tasks after resolving the backlog item
- `notchlet-super` — orchestrates the full cycle including platform discovery

This skill is the entry point to Notchlet-driven development; `notchlet-plan`, `notchlet-implement`, `notchlet-review`, and `notchlet-deliver` assume the platform and item have already been resolved.
