# Phase 2: Spec — Detailed Instructions

## Goal
Generate OpenSpec artifacts: proposal.md, specs/, design.md, tasks.md.

## Trigger
Run `/bloom-spec`

## What Happens
1. Agent explores the codebase
2. Agent checks for supporting docs (requirements.md, project-overview.md, CONTEXT.md) and asks user whether to use them
3. If no requirements doc available, asks: grill first or describe now?
4. Agent runs ambiguity check on requirements
5. Agent asks for change name (must be user-provided, no derivation)
6. Agent runs `openspec new change <name>`
7. Agent loops through artifacts using `openspec instructions` (CLI owns the format)
8. Tasks.md uses standard `- [ ] X.Y Task description` format

## Completion Criterion
- All `applyRequires` artifacts are `done`
