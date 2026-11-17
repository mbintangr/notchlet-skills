# Notchlet-Driven Development Skills Repository

This repository contains OpenCode skills for **Notchlet-driven development**, formatted for [skills.sh](https://www.skills.sh/). Install them with:

```bash
npx skills add mbintangr/notchlet-skills
```

## Skills Included

| Skill | Purpose |
|-------|---------|
| `notchlet-super` | **Meta-skill** — orchestrates the full cycle: setup → discover → plan → implement → review → deliver. Asks for your authorization at each gate. |
| `notchlet-mcp` | Configure opencode.jsonc and verify connection to the remote Notchlet MCP backend |
| `notchlet-platform` | Discover products, resolve backlog items, understand available MCP operations |
| `notchlet-plan` | Create/refine epics, stories, tasks, acceptance criteria, dependencies |
| `notchlet-implement` | Implement backlog items/tickets from discovery through verification |
| `notchlet-review` | Review implemented work, approve or request changes before delivery |
| `notchlet-deliver` | Record evidence and update statuses based on verified completion |

Start with `notchlet-super` for full-cycle development. It drives all other skills interactively. The individual skills are available if you need a single phase only.

These seven skills cover the full Notchlet-driven development loop: **setup → discover → plan → implement → review → deliver**.

Each skill is a directory under `skills/` containing a `SKILL.md` file with YAML frontmatter (`name`, `description`), following the [skills.sh skill format](https://github.com/vercel-labs/skills/blob/main/docs/creating-skills.md).

## Quick Start

### Recommended: Full cycle with notchlet-super

```bash
npx skills add mbintangr/notchlet-skills --skill notchlet-super -a opencode
```

Then tell the agent: *"Use notchlet-super to implement story APP-42 from idea to live."* The agent will walk you through setup, discovery, planning, implementation, review, and delivery — asking for authorization at each gate.

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
├── README.md         # This file
└── skills/
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
