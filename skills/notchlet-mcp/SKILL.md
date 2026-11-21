---
name: notchlet-mcp
description: Set up or diagnose an authenticated Notchlet MCP connection for OpenCode or Claude Code when the required tools are not already available.
---

# Notchlet MCP setup and handoff

Reuse an authenticated Notchlet MCP server already available to the agent. Do not request or create another credential just to repeat setup. The account session used by the Notchlet CLI for management is distinct from any MCP client credential; never copy either credential into project config, a prompt, shell history, or repository files.

## Setup

1. Inspect the current client setup and discover the live Notchlet MCP capabilities/tools first. Do not overwrite existing configuration or assume a server URL, token, scope, or tool name.
2. If authenticated Notchlet tools are already available, call the advertised tools and skip setup.
3. Choose the transport that matches the work:
   - **Offline/local (no account):** `notchlet init --local --client opencode` (or `claude`) installs skills and writes `{"type":"local","command":["notchlet","mcp","serve","--local"]}`. Agents get local authoring, task, verify, and evidence tools without network access.
   - **Connected stdio bridge (default):** `notchlet init --interactive`, or `notchlet mcp configure --client opencode` after login. Writes a bridge entry; the credential stays in the OS credential store.
   - **Direct remote HTTP:** `notchlet mcp configure --client opencode --transport remote --server <url>`. Writes the OpenCode `type: "remote"` entry with `url: "<url>/mcp"` and `headers.Authorization: "Bearer {env:NOTCHLET_MCP_TOKEN}"` (use `--token-env NAME` for a different variable). Set that environment variable in the process that launches the client; `notchlet mcp token --server <url>` prints the stored credential for that purpose (treat it as a secret). For Claude Code the same command writes `type: "http"`.
4. Connected setup creates a scoped credential, stores it in the OS credential store, installs/updates the matching skills, and merges the entry. Account-wide MCP enablement requires a separate explicit confirmation (`--yes` approves it non-interactively).
5. Refresh installed guidance without touching MCP configuration with `notchlet client init <agent> --apply` or `notchlet client update <agent> --apply`. Targets are `opencode` (`.opencode/skills`), `claude` (`.claude/skills`), and `agent` (`.agents/skills`); `agent` is available only through `notchlet client`, not `init --client` or `mcp configure`.
6. If management fails because no account session exists, use `notchlet auth login --server <url>` with a user account session, then rerun setup. A scoped `NOTCHLET_TOKEN` is not an account session and cannot enable MCP or create credentials. Never print or persist credentials, and never place a literal bearer token in project config.
7. Verify with `notchlet mcp status` and authenticated MCP initialization plus `tools/list`; `notchlet doctor` checks the CLI/workspace binding. A failed initialization, missing tools, or authorization error blocks remote operations; continue independent local work and report the failure accurately.

## CLI / MCP handoff

- MCP is for authenticated remote discovery, bounded context, and authorized platform operations. Use only tool names and argument schemas actually advertised by the connected server. A local server advertises `notchlet_*` tools for changes, tasks, verify, and evidence.
- The CLI is for workspace setup, local change/task authoring, and authenticated platform workflows:
  - `notchlet change new <name> --quick`
  - `notchlet change update <change> --intent <text> --outcome <text> --capability-impact none --capability-rationale <text>`
  - `notchlet change update <change> <kind> <item-id> ...` (in-place item edit; preserves unrelated fields and task progress)
  - `notchlet change add <change> requirement|check|scenario|decision|task ...`
  - `notchlet change apply <change> --file <plan.json>` (atomic ordered batch, one revision bump, full rollback on failure)
  - `notchlet change remove <change> <kind> <item-id>`
  - `notchlet change render <change> --check|--write [--force]` (declared proposal/spec artifacts)
  - Authoring commands accept `--revision` to reject a stale write with `revision_conflict` before writing.
  - `notchlet task start|complete <task> --change <change>` (revision defaults to current)
  - `notchlet verify <change> --record`
  - `notchlet evidence list --change <change>`
  - `notchlet status <change>`, `notchlet validate <change>`, `notchlet instructions <change> --action <plan|revise|implement|verify|resume>`
- Prefer these commands over hand-editing `tasks.json`; they enforce references, revisions, and lifecycle ordering.
- For backlog-to-local handoff, resolve the item over MCP, then use `notchlet backlog import <key-or-title>` from the initialized, correctly bound workspace when a local change is needed. Check CLI help and returned state; do not guess identifiers, create duplicate changes, or claim the import succeeded without confirmation.
- Never verify MCP with unauthenticated `GET /mcp`. Do not probe endpoints directly; use authenticated client initialization and `tools/list`.

## Troubleshooting

- **Tools unavailable:** inspect existing authenticated client setup and advertised tools; run `notchlet mcp configure --client <name>` only if MCP setup is actually missing. Without an account, configure the local server instead.
- **Remote transport unauthorized:** confirm the environment variable referenced by `headers` (default `NOTCHLET_MCP_TOKEN`) is set in the client's environment; do not paste the token into prompts or config.
- **No account session for credential management:** explain that the CLI account login is needed for management, separate from an API token, and wait for the user to complete login.
- **Config conflict or malformed setup:** preserve the file, show the conflict, and get authorization before editing.
- **Inspect/remove setup:** use `notchlet mcp status` to inspect the local entry and credential state; use `notchlet mcp disconnect` or `notchlet mcp revoke` only when the user requests removal/revocation.

## When not to use

- The authenticated MCP client already initializes and advertises the needed tools.
- The task is local-only and does not need Notchlet platform access.

## Related skills

- `notchlet-platform` — discover products and resolve existing backlog items
- `notchlet-super` — route the complete workflow and hand off between MCP and CLI
