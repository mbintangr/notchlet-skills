---
name: notchlet-init
description: Legacy compatibility skill for explicitly requested Notchlet workspace initialization or context-file setup; not required before using notchlet-super.
---

# Notchlet Init — optional compatibility helper

This legacy skill is optional. Do not run it automatically before `notchlet-super`, and do not create AGENTS.md or PROMPT.md as a prerequisite for local Notchlet work.

## When to use

- The user explicitly asks to initialize a local Notchlet workspace. Prefer the shipped CLI command `notchlet init` in the intended checkout.
- The user explicitly asks to create or revise AGENTS.md/PROMPT.md context files. Preview the proposed content first; preserve existing user edits and only write after approval.
- Connected `notchlet init --interactive` can optionally configure MCP for the selected OpenCode or Claude Code client through the supported setup flow. This legacy context-file skill does not alter credentials or client connection configuration. Reuse an authenticated connection when present; preserve existing config/trust settings and never put a static project token into client files or generated context.

## Safe behavior

1. Confirm the checkout and target directory before writing.
2. Inspect for existing `.notchlet`, AGENTS.md, and PROMPT.md files. Never overwrite existing work silently.
3. Draft AGENTS.md from verified repository instructions, commands, conventions, and test scripts; draft PROMPT.md from verified project structure and technical context. Do not invent project facts or credentials.
4. Present a preview of new or changed files and obtain approval before writing. For existing files, preserve manually edited content and reconcile sections rather than replacing the document.
5. Keep initialization local. Do not add tokens, personal data, or machine-specific paths to repository files.
6. For an initialized workspace, skip setup and route to the requested phase with `notchlet-super` or the relevant phase skill.
7. For connected diagnostics, initialize the authenticated MCP client and use `tools/list`, or run `notchlet doctor` for the authenticated CLI/workspace binding. Never use an unauthenticated HTTP GET to `/mcp` as a connection test.
8. Local-only `notchlet init` does not configure MCP. Connected interactive init offers MCP setup separately. For the local handoff, reuse the bound workspace, import the confirmed backlog item by key/title when needed, then inspect CLI status/instructions and manage local change tasks through supported commands. Keep MCP for authenticated remote discovery and operations; never infer success from initiating a command.

## Related skills

- `notchlet-super` — state-aware routing that skips valid setup
- `notchlet-mcp` — authenticated MCP connection setup when needed
- `notchlet-platform` — resolve an unknown product or backlog item
