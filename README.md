# Notchlet-Driven Development Skills Repository

This repository contains state-aware **Notchlet-driven development** skills for OpenCode and Claude Code, formatted for [skills.sh](https://www.skills.sh/). Install the router and its phase skills with:

```bash
npx skills add mbintangr/notchlet-skills
```

## Skills Included

| Skill | Purpose |
|-------|---------|
| `notchlet-init` | Optional legacy helper for context-file generation; never run automatically |
| `notchlet-super` | **State-aware router** — resumes current work, skips valid setup, and routes to the requested phase. |
| `notchlet-mcp` | Configure and verify an authenticated Notchlet MCP connection when needed |
| `notchlet-platform` | Discover products, resolve backlog items, understand available MCP operations |
| `notchlet-plan` | Create/refine epics, stories, tasks, acceptance criteria, dependencies |
| `notchlet-implement` | Implement backlog items/tickets from discovery through verification |
| `notchlet-review` | Review implemented work, approve or request changes before delivery |
| `notchlet-deliver` | Record evidence and update statuses based on verified completion |

Start with `notchlet-super` for a full-cycle request, or use one phase skill directly. The router reuses a valid workspace, item, and change; it does not initialize context files, restart setup, or re-plan existing work by default. Use the `notchlet init` CLI only when local workspace setup is needed and authorized.

The skills cover the Notchlet-driven development loop: **discover → plan → implement → review → deliver**, with setup only when needed.

## Legacy aliases

| Legacy name | Current route |
| --- | --- |
| `notchlet-development` | `notchlet-super` for the full-cycle request; use the individual phase skill for a single phase. |
| `notchlet-backlog-planning` | `notchlet-plan` |
| `notchlet-delivery` | `notchlet-deliver` |
| `notchlet-init` | Optional legacy context helper. Local workspace creation is `notchlet init`; MCP connection setup is `notchlet-mcp`. |
| `notchlet-n8n-migration` | Specialized workflow: retain an installed copy; it is not part of the core Notchlet skill set. |

```text
┌────────────┐    ┌─────────────┐    ┌────────┐    ┌────────────┐    ┌────────┐    ┌─────────┐
│ Current    │───▶│ Requested   │───▶│ Plan   │───▶│ Implement /│───▶│ Review │───▶│ Deliver │
│ state      │    │ phase       │    │ if     │    │ verify     │    │ if     │    │ if      │
│            │    │             │    │ needed │    │            │    │ needed │    │ eligible│
└────────────┘    └─────────────┘    └────────┘    └────────────┘    └────────┘    └─────────┘
```

Each skill is a directory under `skills/` containing a `SKILL.md` file with YAML frontmatter (`name`, `description`), following the [skills.sh skill format](https://github.com/vercel-labs/skills/blob/main/docs/creating-skills.md).

## Quick Start

### Recommended: state-aware routing

```bash
npx skills add mbintangr/notchlet-skills --skill notchlet-super
```

Then tell the agent: *"Use notchlet-super to continue the active Notchlet change."* It inspects current state and resumes the requested phase. Setup, planning, review, and delivery are performed only when needed, with authorization required for external side effects.

### Install a single skill

```bash
npx skills add mbintangr/notchlet-skills --skill notchlet-implement -a opencode
```

### Install all skills

```bash
npx skills add mbintangr/notchlet-skills
```

### Use without installing

```bash
npx skills use mbintangr/notchlet-skills@notchlet-super --agent opencode
```

## File Structure

```
notchlet-skills/
├── AGENTS.md         # Agent operating instructions
├── PROMPT.md         # Project context
├── README.md         # This file
└── skills/
    ├── notchlet-init/
    │   └── SKILL.md    # Workspace bootstrap: context files + MCP config
    ├── notchlet-mcp/
    │   └── SKILL.md
    ├── notchlet-platform/
    │   └── SKILL.md
    ├── notchlet-plan/
    │   └── SKILL.md
    ├── notchlet-implement/
    │   └── SKILL.md
    ├── notchlet-review/
    │   └── SKILL.md
    ├── notchlet-deliver/
    │   └── SKILL.md
    └── notchlet-super/
        └── SKILL.md   # Meta-skill: full-cycle orchestrator
```

## License

MIT
