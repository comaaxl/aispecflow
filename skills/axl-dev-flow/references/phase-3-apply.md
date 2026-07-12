# Phase 3: Apply - Detailed Instructions

## Goal
Implement all tasks. Choice of TDD or straight implementation, with optional
task-level review and git checkpoint commits.

## Trigger
Run `/grow-apply`

## What Happens
1. **Step 0 - Git state check + resume check**: first, if an apply state file
   already exists, cross-check it against tasks.md and git log - if inconsistent
   or incomplete (previous apply interrupted), stop and ask the user how to
   resume (continue from first incomplete task / re-review last task / abort).
   Then detect git availability and working-tree cleanliness. Clean tree ->
   record BASE, write apply state file. Dirty tree -> stop and ask the user
   (commit / stash / abort). No git -> stop and ask (continue degraded /
   `git init`).
2. Agent selects the change
3. Agent runs `openspec instructions apply` to get context
4. Agent asks the user three questions, one at a time:
   - TDD (RED->GREEN->REFACTOR) or straight implementation?
   - Task-level review during implementation? (default on; forced off if no git)
   - How to fix review findings? (asked once, reused for every task; only if
     task-level review is on; default: fix in conversation)
5. Agent implements tasks one at a time, marking each immediately (exact line
   replacement - no batch regex)
6. After each task: checkpoint commit (`task(X.Y): ...`, updates state file
   `head`), then optional task-level review (hands control to `/prune-review`
   at task level). Fixes commit as `fix(review-task-X): ...`. Re-review is the
   user's call, not automatic.

## TDD Iron Law
NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST.

## Completion Criterion
- All tasks checked `[x]`
- All tests green
- Apply state file recorded `base` (or `no-git`), `head` updated through
  checkpoint commits

## Pause Conditions
- Task is unclear -> ask
- Design issue revealed -> suggest updating artifacts
- Error/blocker -> report and wait

## No-Git Degradation
When there is no git: implementation still works, but task-level review is
unavailable, change-level review degrades to the whole working tree, and
progress recovery relies only on tasks.md checkboxes. Recommend `git init`.
