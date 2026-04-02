# aloop-spine

Operational architecture spine for ALoop. This repo is the single source of truth for AI coding agents working across ALoop repositories.

## What This Is

This is NOT a documentation archive. It is the **operational control layer** that AI coding agents (Claude Code, Cursor, Copilot, Cline) read before making changes to any ALoop repo. It defines what the system is, how it's built, and what conventions to follow.

## Numbered Architecture Files

| File | Topic | Description |
|------|-------|-------------|
| 00 | Product Context | What ALoop is, market, users, value prop |
| 01 | Tech Stack | Languages, frameworks, libraries, versions |
| 02 | Architecture | SPA structure, feature modules, data flow |
| 03 | UI Components | Component families, shadcn inventory, card system |
| 04 | Routing | All routes, lazy loading, auth guards |
| 05 | API Contracts | Endpoints, clients, auth flow, socket events |
| 06 | State Management | Context API, React Query, workspace resolution |
| 07 | Integrations | Platform connections, OAuth flows |
| 08 | Design System | Tokens, colors, typography, dark mode |
| 09 | Permissions & RBAC | Roles, workspaces, feature flags |
| 10 | Dev Workflow | Git conventions, PRs, testing, branches |

## How To Use

**AI agents:** Your repo's CLAUDE.md or AGENTS.md will tell you to read this spine. Start with 00, then read files relevant to your task.

**Humans:** Browse the numbered files for a complete picture of the ALoop architecture. Each file is self-contained.

## Audit Artifacts

The `audit/` folder contains the raw outputs from the three-agent audit process that generated these files:
- `outside-in/` - Architecture derived from existing documentation
- `inside-out/` - Architecture derived from direct code inspection
- `consensus/` - Gap analysis comparing the two, plus the merged result
