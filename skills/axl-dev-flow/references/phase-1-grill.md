# Phase 1: Grill — Detailed Instructions

## Goal
Produce `docs/requirements.md` — a validated, unambiguous requirements document.

## Trigger
run `/seed-grill` 

## What Happens

1. Agent explores the codebase directly — code is the primary source of truth
2. Agent reads `CONTEXT.md` for domain terms (if exists)
3. Agent reads relevant ADRs for settled decisions
4. Agent reads `docs/project-overview.md` if exists but verifies against code
5. Agent interviews user one question at a time
6. Agent writes `CONTEXT.md` updates and ADRs inline
7. Agent produces `docs/requirements.md` at the end
8. User confirms the document

## Completion Criterion
- User confirms `docs/requirements.md` is complete and correct
