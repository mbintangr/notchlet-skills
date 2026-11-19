---
name: notchlet-mcp
description: Set up or diagnose an authenticated Notchlet MCP connection for OpenCode or Claude Code when the required tools are not already available.
---

# Notchlet MCP setup and handoff

Reuse an authenticated Notchlet MCP server already available to the agent. Do not request or create another credential just to repeat setup. The account session used by the Notchlet CLI for management is distinct from any MCP client credential; never copy either credential into project config, a prompt, shell history, or repository files.

## Setup

1. Inspect the current client setup and discover the live Notchlet MCP capabilities/tools first. Do not overwrite existing configuration or assume a server URL, token, scope, or tool name.
2. If authenticated Notchlet tools are already available, call the advertised tools and skip setup.
3. For a new or existing checkout that needs setup, run `notchlet init --interactive`, choose OpenCode or Claude Code, and select MCP setup. Connected setup creates a scoped credential, stores it in the OS credential store, installs/updates the matching skills, and merges the local bridge entry into the selected project's client config. Account-wide MCP enablement requires a separate explicit confirmation. Local-only init stays offline.
4. If management fails because no account session exists, use `notchlet auth login --server <url>` with a user account session, then rerun init. A scoped `NOTCHLET_TOKEN` is not an account session and cannot enable MCP or create credentials. Never print or persist credentials.
5. Verify with `notchlet mcp status` and authenticated MCP initialization plus `tools/list`; `notchlet doctor` checks the CLI/workspace binding. A failed initialization, missing tools, or authorization error blocks remote operations; continue independent local work and report the failure accurately.

## CLI / MCP handoff

- MCP is for authenticated remote discovery, bounded context, and authorized platform operations. Use only tool names and argument schemas actually advertised by the connected server.
- The CLI is for workspace setup, local change/task state, and authenticated platform workflows. In the intended checkout, use `notchlet status <change>`, `notchlet instructions <change> --action <plan|revise|implement|verify|resume>`, `notchlet validate <change>`, and `notchlet change list` as applicable. Interactive `notchlet init` offers MCP setup; it is not a reason to reset or replace a valid binding.
- For backlog-to-local handoff, resolve the item over MCP, then use `notchlet backlog import <key-or-title>` from the initialized, correctly bound workspace when a local change is needed. Check CLI help and returned state; do not guess identifiers, create duplicate changes, or claim the import succeeded without confirmation.
- Never verify MCP with unauthenticated `GET /mcp`. Do not probe endpoints directly; use authenticated client initialization and `tools/list`.

## Troubleshooting

- **Tools unavailable:** inspect existing authenticated client setup and advertised tools; run `notchlet init --interactive` only if MCP setup is actually missing.
- **Unauthorized / expired:** ask the user to re-authenticate through the supported account/client flow; never request a token in chat.
- **No account session for credential management:** explain that the CLI account login is needed for management, separate from an API token, and wait for the user to complete login.
- **Config conflict or malformed setup:** preserve the file, show the conflict, and get authorization before editing.
- **Inspect/remove setup:** use `notchlet mcp status` to inspect the local entry and credential state; use `notchlet mcp disconnect` or `notchlet mcp revoke` only when the user requests removal/revocation.

## When not to use

- The authenticated MCP client already initializes and advertises the needed tools.
- The task is local-only and does not need Notchlet platform access.

## Related skills

- `notchlet-platform` — discover products and resolve existing backlog items
- `notchlet-super` — route the complete workflow and hand off between MCP and CLI
