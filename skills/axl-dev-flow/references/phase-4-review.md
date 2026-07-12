# Phase 4: Review - Detailed Instructions

## Goal
Code review via an independent subagent. Catch issues before archiving.
Prevents self-review bias.

## Trigger
Run `/prune-review`

## What Happens
1. Agent asks the user what level to review (default: change-level):
   - **Change-level (default)** - review all of this OpenSpec change's changes
     (range: base..HEAD from the apply state file)
   - **Project-level** - a health check on the whole codebase (full read, no
     diff)
   - **Task-level** - review a single task (usually triggered automatically
     during `/grow-apply`; pick manually only to re-review a specific task)
2. Agent derives the git range internally from the apply state file - never
   asks the user for git commands. No-git: task-level unavailable, change-level
   degrades to whole working tree, project-level unaffected.
3. Agent prepares context (description, proposal/specs/tasks for change-level;
   project root for project-level)
4. Agent writes the diff to a file (change/task-level) and dispatches an
   independent, read-only reviewer subagent with the level-appropriate template.
   Project-level gets project root path instead of a diff.
5. Evaluator verifies each issue before acting. ⚠️ cannot-verify-from-diff
   items (change-level) are resolved by the controller, who holds cross-task
   context.
6. Critical + Important issues fixed (inline or via fix subagent, per the
   apply state file's `fix_mode`); re-review is the user's call, not automatic.

## Completion Criterion
- All Critical issues fixed
- All Important issues fixed
- Reviewer verdict is "Approved" or "Approved with fixes"

## Note on Levels
Task-level review is normally done during `/grow-apply` (Phase 3) if the user
enabled it. Phase 4's default is change-level - the whole-change review before
archive. Project-level is an optional periodic health check.
