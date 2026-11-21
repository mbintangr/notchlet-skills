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
- skills/notchlet-init/SKILL.md — optional, user-requested context-file setup
- skills/notchlet-mcp/SKILL.md — configure an authenticated MCP connection and verify advertised tools
- skills/notchlet-platform/SKILL.md — discover products, resolve backlog items; the item key drives issue, branch, and PR
- skills/notchlet-plan/SKILL.md — grill requirements, then create/refine epics, stories, tasks with Definition of Ready
- skills/notchlet-implement/SKILL.md — create/link the Git issue, create and checkout the branch, implement, check, commit, push
- skills/notchlet-review/SKILL.md — review the PR against the backlog description and acceptance criteria
- skills/notchlet-deliver/SKILL.md — open the PR, run review, drive fixes, merge, record evidence, set DONE
- skills/notchlet-super/SKILL.md — full-cycle orchestrator; holds the consolidated CLI tracking reference

## Base Flow
1. Plan and grill the request until requirements are unambiguous.
2. Create epic → stories → tasks on Notchlet with a Definition of Ready.
3. Implement per item: create/link Git issue → create branch → checkout → develop → commit → push.
4. Deliver: open PR → review against the backlog description → fix/commit/push → merge.
5. Done: record evidence, set backlog status DONE, sync the Git issue.

The Notchlet CLI (change/task/verify/evidence/status/sync/checkpoint) is the primary local tracking layer when available; it complements the Git flow rather than replacing it.

## Capability Modes
- Connected — authenticated Notchlet MCP is available: the platform is authoritative for requirements/status and the CLI is the primary local tracking layer.
- Local — no account, but the CLI/offline server works: the CLI is the primary workflow and platform-only steps are deferred as `sync pending`.
- Neither — repository-only work; tracking unavailable.
Prefer connected whenever present; never present local evidence or closure as a remote update.

## Conventions
- YAML frontmatter (name, description) at the top of each SKILL.md
- Conventional commits (feat:, fix:, chore:, docs:)
- Do not commit secrets or keys; use environment variables

## Context Files
- AGENTS.md — agent operating instructions tuned for Notchlet-driven development
- PROMPT.md — this file, project context
- OpenCode `mcp.servers` configuration and Claude Code `.mcp.json` are client-managed setup; use the authenticated, currently supported Notchlet flow and never store a static project token in either file.
