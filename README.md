# Notchlet-Driven Development Skills Repository

This repository contains **Notchlet-driven development** skills for OpenCode and Claude Code, formatted for [skills.sh](https://www.skills.sh/). Install them with:

```bash
npx skills add mbintangr/notchlet-skills
```

## Skills Included

| Skill | Purpose |
|-------|---------|
| `notchlet-super` | **Full-cycle orchestrator** — grill and plan → create backlogs → issue/branch/develop/push → PR/review/fix/merge → done. Asks for authorization at each gate. |
| `notchlet-init` | Optional legacy helper for context-file generation; never run automatically |
| `notchlet-mcp` | Configure and verify an authenticated Notchlet MCP connection when needed |
| `notchlet-platform` | Discover products and resolve backlog items; the item key drives the issue, branch, and PR |
| `notchlet-plan` | Grill requirements, then create/refine epics, stories, tasks with a Definition of Ready |
| `notchlet-implement` | Create/link the Git issue, create and checkout the branch, implement, check, commit, push |
| `notchlet-review` | Review the PR against the backlog description and acceptance criteria |
| `notchlet-deliver` | Open the PR, run review, drive fixes, merge, record evidence, set DONE |

Start with `notchlet-super` for a full-cycle request, or use one phase skill directly.

## The base flow

```text
┌──────────────┐   ┌──────────┐   ┌───────────────┐   ┌──────────────┐   ┌──────┐
│ 1. Plan &    │──▶│ 2. Create│──▶│ 3. Implement  │──▶│ 4. Deliver   │──▶│ 5.   │
│    grill     │   │  backlogs│   │ issue → branch│   │ PR → review  │   │ Done │
│              │   │          │   │ → code → push │   │ → fix → merge│   │      │
└──────────────┘   └──────────┘   └───────────────┘   └──────────────┘   └──────┘
```

1. **Plan & grill** — interrogate the request until requirements are unambiguous, then plan.
2. **Create backlogs** — epic → stories → tasks on Notchlet with measurable acceptance criteria and a Definition of Ready.
3. **Implement** — create/link the Git issue, create the `<type>/<key>-<slug>` branch, checkout, develop, run checks, commit, push.
4. **Deliver** — open the PR, review it against the backlog description, fix/commit/push if changes are requested, then merge.
5. **Done** — record evidence, set the backlog item to DONE, sync the Git issue.

Authorization is required before creating the remote issue and branch, pushing, opening a PR, merging, or setting DONE.

## Capability modes

Every skill determines the mode before acting:

- **Connected** — authenticated Notchlet MCP tools are available. The platform is authoritative for requirements and status; the CLI is the primary local tracking layer and stays linked through import/sync.
- **Local** — no account, but the CLI/offline server is available. The CLI is the primary workflow; platform-only steps (backlog creation, Notchlet issue linking, remote status) are deferred as `sync pending`.
- **Neither** — repository-only work; tracking is unavailable and reported as a blocker.

Prefer connected whenever present; never silently fall back to local because local always works, and never present local evidence or closure as a remote update. Detection lives in `notchlet-mcp`.

## CLI tracking layer

The Notchlet CLI (`notchlet change`, `task`, `verify`, `evidence`, `checkpoint`, `sync`) is the primary local tracking layer whenever available: in connected mode it mirrors the platform backlog and syncs; in local mode it is the only tracking store until sync is possible. It complements the Git flow rather than replacing it. The consolidated command reference lives in `skills/notchlet-super/SKILL.md`.

## Legacy aliases

| Legacy name | Current route |
| --- | --- |
| `notchlet-development` | `notchlet-super` for the full-cycle request; use the individual phase skill for a single phase. |
| `notchlet-backlog-planning` | `notchlet-plan` |
| `notchlet-delivery` | `notchlet-deliver` |
| `notchlet-init` | Optional legacy context helper. Local workspace creation is `notchlet init`; MCP connection setup is `notchlet-mcp`. |
| `notchlet-n8n-migration` | Specialized workflow: retain an installed copy; it is not part of the core Notchlet skill set. |

Each skill is a directory under `skills/` containing a `SKILL.md` file with YAML frontmatter (`name`, `description`), following the [skills.sh skill format](https://github.com/vercel-labs/skills/blob/main/docs/creating-skills.md).

## Quick Start

### Recommended: full cycle with notchlet-super

```bash
npx skills add mbintangr/notchlet-skills --skill notchlet-super
```

Then tell the agent: *"Use notchlet-super to take APP-42 from plan to merged."* The agent will grill the requirements, create the backlog, create the issue and branch, implement, push, open a PR, review it against the backlog description, fix anything rejected, merge, and set the item to DONE — pausing for authorization at each gate.

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

### Install through the Notchlet CLI

```bash
notchlet client init opencode --apply    # targets: opencode, claude, agent
notchlet client update opencode --apply  # refresh installed guidance later
```

`notchlet init --client opencode|claude` also installs skills together with the local MCP configuration; `notchlet client` handles skills only (including the generic `.agents/skills` target).

## File Structure

```
notchlet-skills/
├── AGENTS.md         # Agent operating instructions
├── PROMPT.md         # Project context
├── README.md         # This file
└── skills/
    ├── notchlet-init/
    │   └── SKILL.md    # Optional legacy context helper
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
        └── SKILL.md   # Full-cycle orchestrator
```

## License

MIT
