# Phase 3: Apply - Detailed Instructions

## Goal
Implement all tasks. Choice of TDD or straight implementation, with always-on
self-review, git checkpoint commits, and optional subagent review.

## Trigger
Run `/grow-apply`

## What Happens
1. **Step 0 - Git state check + resume check**: first, if an apply state file
   already exists, cross-check it (base/head/last_completed_task) against
   tasks.md and git log - if inconsistent or incomplete (previous apply
   interrupted), stop and ask the user how to resume (continue from first
   incomplete task / re-review last task / abort). Then detect git availability
   and working-tree cleanliness. Clean tree -> record BASE, write apply state
   file. Dirty tree -> stop and ask the user (commit / stash / abort). No git ->
   stop and ask (continue degraded / `git init`).
2. Agent selects the change
3. Agent runs `openspec instructions apply` to get context
4. Agent asks the user three questions, one at a time:
   - TDD (RED->GREEN-REFACTOR) or straight implementation?
   - Subagent review depth? (`every` / `final` [default] / `none`; `every`
     unavailable without git) - self-review is always on regardless
   - How to fix subagent review findings? (asked once, reused; only when
     subagent review is `every` or `final`; skipped for `none`; default: fix in
     conversation)
5. Agent implements tasks one at a time
6. After each task's tests pass: **self-review** (checklist, inline fixes,
   re-test) -> **checkpoint commit** (`task(X.Y): ...`, updates `head`) ->
   optional **subagent review** (only if `per_task_review: every`) -> mark
   `[x]` (advances `last_completed_task`). Fixes commit as
   `fix(review-task-X): ...`. Re-review is the user's call, not automatic.

## TDD Iron Law
NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST.

## Completion Criterion
- All tasks checked `[x]`
- All tests green
- Apply state file recorded `base` (or `no-git`), `head` and
  `last_completed_task` updated through the run

## Pause Conditions
- Task is unclear -> ask
- Design issue revealed -> suggest updating artifacts
- Error/blocker -> report and wait

## No-Git Degradation
When there is no git: implementation still works, self-review still works, but
subagent per-task review (`every`) is unavailable, change-level subagent review
degrades to the whole working tree, and progress recovery relies only on
tasks.md checkboxes + `last_completed_task`. Recommend `git init`.
