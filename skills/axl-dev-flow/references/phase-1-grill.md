# Phase 1: Grill - Detailed Instructions

## Goal
Produce the document(s) the user asked for - validated, unambiguous, professionally structured.

## Trigger
run `/seed-grill`

## What Happens

1. Agent explores the codebase directly - code is the primary source of truth
2. Agent reads `CONTEXT.md` for domain terms (if exists)
3. Agent reads relevant ADRs for settled decisions
4. Agent reads `docs/project-overview.md` if exists but verifies against code
5. Agent determines user intent - what document(s) do they want this round?
6. Agent interviews user one question at a time, adapting depth/focus to the target document type
7. Agent writes `CONTEXT.md` updates and ADRs inline
8. Agent produces the document(s) the user asked for under `docs/`
9. User confirms the document(s)

## Completion Criterion
- User confirms the produced document(s) are complete and correct
