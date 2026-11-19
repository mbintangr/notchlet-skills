---
name: notchlet-mcp
description: Use when configuring opencode.jsonc to connect to the remote Notchlet MCP backend and verifying the connection before using Notchlet-driven development skills.
---

# Notchlet MCP Setup

Use this skill to configure OpenCode to connect to the existing Notchlet MCP backend so that `notchlet-platform`, `notchlet-plan`, `notchlet-implement`, `notchlet-review`, and `notchlet-deliver` can work against a live platform.

## Prerequisites

- **OpenCode** installed
- A valid MCP bearer token for the Notchlet backend

## Step 1: Configure opencode.jsonc

Edit your OpenCode configuration file (typically at `~/.config/opencode/opencode.jsonc` or `~/.opencode/opencode.jsonc`):

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "notchlet": {
      "type": "remote",
      "url": "https://pg.mbintangr.com/mcp",
      "oauth": false,
      "headers": {
        "Authorization": "Bearer your-mcp-token"
      }
    }
  }
}
```

Replace `your-mcp-token` with the MCP bearer token for the Notchlet backend. The URL points to the hosted platform at `pg.mbintangr.com`.

## Step 2: Verify the connection

Quit and restart OpenCode so the authenticated MCP client can initialize and list the server's advertised tools. An unauthenticated HTTP `GET /mcp` is not a valid connection test: this server uses authenticated Streamable HTTP `POST` requests. If the Notchlet CLI is installed and the current directory is an initialized workspace, run `notchlet doctor` for authenticated capabilities and identity diagnostics.

## Troubleshooting

- **MCP connection fails:** Verify the user-managed credential/configuration and restart OpenCode. Use an authenticated MCP initialization or `tools/list` request; do not use unauthenticated GET probes.
- **Skills not showing:** Quit and restart OpenCode completely after configuring `opencode.jsonc`.
- **No MCP token:** Ask the platform owner for a Notchlet MCP bearer token. Never put the token in a repository file, shell history, or command-line argument.

## When to use

- Before using any other Notchlet skill for the first time on a new machine or project.
- When `notchlet_workspace_context` or `backlog_development_context` tools are unavailable.

## When NOT to use

- When the authenticated MCP client already initializes and lists the needed tools, or `notchlet doctor` reports a healthy binding.
- When you only need to plan, implement, or deliver — use the individual skill instead.

## Related skills

- `notchlet-platform` — discover products and resolve backlog items (requires MCP connection)
- `notchlet-super` — orchestrates the full cycle including MCP setup
