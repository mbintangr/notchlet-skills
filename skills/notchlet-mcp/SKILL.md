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

Quit and restart OpenCode so the `notchlet_*` MCP tools are advertised. Then run:

```bash
curl https://pg.mbintangr.com/mcp
```

Should return an MCP server initialization response.

## Troubleshooting

- **MCP connection fails:** Verify the `Authorization` header matches the token expected by the backend, and that `curl https://pg.mbintangr.com/mcp` returns a response.
- **Skills not showing:** Quit and restart OpenCode completely after configuring `opencode.jsonc`.
- **No MCP token:** Ask the platform owner for a Notchlet MCP bearer token.

## When to use

- Before using any other Notchlet skill for the first time on a new machine or project.
- When `notchlet_workspace_context` or `backlog_development_context` tools are unavailable.
