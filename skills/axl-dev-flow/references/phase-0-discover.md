# Phase 0: Discover — Detailed Instructions

## Goal
Produce `docs/project-overview.md` — a comprehensive project overview that any agent can read to understand the codebase.

## Trigger
Run `/terrain-scan`

## What the Agent Does
1. Reads existing docs (README, CONTEXT, AGENTS, CLAUDE)
2. Explores the codebase directly — code is the ultimate source of truth
3. Dispatches subagents to explore architecture, modules, domain, and conventions
4. Merges reports into a single `docs/project-overview.md`
5. Presents to user for confirmation

## Completion Criterion
- `docs/project-overview.md` exists
- User confirms it's accurate

## Skip Conditions
- `docs/project-overview.md` exists AND user has verified it matches the current codebase
- If in doubt, re-run discovery
