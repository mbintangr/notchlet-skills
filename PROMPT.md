# Project Context — notchlet-skills

## Overview
OpenCode skills for Notchlet-driven development, formatted for skills.sh.

## Tech Stack
- Runtime: none (skills are markdown/YAML)
- Language: none (skills are markdown/YAML)
- Package Manager: npx (skills CLI)
- Framework: skills.sh skill format
- Test Framework: none

## Project Structure
- skills/ — skill directories, each containing a SKILL.md
- README.md — repository overview and installation instructions
- skills-lock.json — lock file with skill sources, commits, paths, and hashes
- AGENTS.md — agent operating instructions
- PROMPT.md — this file, project context
- .git/ — git metadata

## Key Files
- skills/notchlet-init/SKILL.md — workspace bootstrap: context files + Notchlet-optimized agent instructions
- skills/notchlet-mcp/SKILL.md — configure opencode.jsonc and verify MCP connection
- skills/notchlet-platform/SKILL.md — discover products, resolve backlog items
- skills/notchlet-plan/SKILL.md — create/refine epics, stories, tasks with Definition of Ready
- skills/notchlet-implement/SKILL.md — implement backlog items/tickets
- skills/notchlet-review/SKILL.md — review PR/MR against acceptance criteria
- skills/notchlet-deliver/SKILL.md — record evidence, update status, sync issues
- skills/notchlet-super/SKILL.md — meta-skill: full-cycle orchestrator

## Conventions
- YAML frontmatter (name, description) at the top of each SKILL.md
- Conventional commits (feat:, fix:, chore:, docs:)
- Do not commit secrets or keys; use environment variables

## Context Files
- AGENTS.md — agent operating instructions tuned for Notchlet-driven development
- PROMPT.md — this file, project context
- opencode.jsonc — OpenCode configuration including Notchlet MCP (user-local, not in repo)
