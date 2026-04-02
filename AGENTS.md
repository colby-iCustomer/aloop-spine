# ALoop Architecture Spine

This repo contains the operational architecture docs for all ALoop repositories.
Read the numbered files (00-10) for full context before making changes to any ALoop repo.

## For AI Coding Agents

- Read 00-product-context.md first for orientation
- Then read files relevant to your task (see index below)
- These docs are the SSOT for architectural decisions
- Do NOT modify these files from code repos - changes go through the spine repo directly

## File Index

00 - Product Context (what ALoop is)
01 - Tech Stack (languages, frameworks, libraries)
02 - Architecture (structure, patterns, data flow)
03 - UI Components (component library, families, inventory)
04 - Routing (all routes, guards, lazy loading)
05 - API Contracts (endpoints, clients, socket events)
06 - State Management (context, queries, caching)
07 - Integrations (platforms, OAuth, connected accounts)
08 - Design System (tokens, colors, typography, dark mode)
09 - Permissions & RBAC (roles, workspaces, feature flags)
10 - Dev Workflow (git, PRs, testing, branches)

## Referencing From Code Repos

Add this to your repo's CLAUDE.md, AGENTS.md, or .cursor/rules:

    ## Architecture Context
    Read the ALoop Spine repo for shared architecture docs:
    Path: n:/Github/repos/icustomer/aloop-spine/
    Start with 00-product-context.md, then read files relevant to your task.
