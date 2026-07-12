# Changelog

All notable changes to this project will be documented in this file.

## [1.1.0] - 2026-07-12

### Added
- `grow-apply`: git integration - records a base commit and per-task checkpoint commits in an apply state file (`openspec/changes/<change>/.aispecflow-apply-state.md`)
- `grow-apply`: optional task-level review during implementation (default on when git is available); hands control to `/prune-review` after each task's checkpoint commit
- `grow-apply`: three sequential startup questions (implementation mode, task-level review, fix mode) with plain-language explanations
- `prune-review`: three review levels - task / change (default) / project - with git ranges derived internally from the apply state file
- `prune-review`: three reviewer templates - `task-reviewer.md`, `change-reviewer.md` (with ⚠️ cannot-verify-from-diff mechanism), `project-reviewer.md` (full-read health check)
- `prune-review`: `fix-agent.md` template for batch-fixing review findings via a subagent (when `fix_mode: subagent`)
- Reviewer inputs go through file paths (diff written to a file), not pasted into the conversation

### Changed
- `prune-review`: replaced the 5 git-mechanics scope options with 3 intent-based levels; users no longer pick git commands
- `harvest-archive`: Step 4 review check is now a soft, non-blocking reminder instead of a gate
- `axl-dev-flow`: Phase 3/4 references updated for git state check, checkpoint commits, and three-level review
- Commit discipline: append-only throughout (no amend, no default squash); re-review after fixes is the user's call, not automatic

### Removed
- `prune-review/references/code-reviewer.md` (replaced by `change-reviewer.md`)

### Notes
- Works without git: implementation proceeds, but task-level review is unavailable and change-level review degrades to the whole working tree; project-level review is unaffected
- Worktree-transparent: users can self-manage a git worktree and run the full flow inside it, then merge back to main (merge, not rebase, by default)

## [1.0.0] - 2026-07-06

### Added
- Initial release with 8 skills covering full development lifecycle
- `terrain-scan`: multi-agent project scanning → `docs/project-overview.md`
- `seed-grill`: relentless requirements interview with inline CONTEXT.md and ADR generation
- `bloom-spec`: OpenSpec propose with context discovery, ambiguity check, and artifact generation
- `grow-apply`: TDD or straight implementation (user choice) with per-task discipline
- `prune-review`: independent subagent code review with scope selection
- `harvest-archive`: spec sync, requirements archival, change archiving
- `renew-docs`: change-based or full documentation refresh with backup
- `axl-dev-flow`: orchestrator connecting all phases
- Dual platform support: Claude Code and Code
