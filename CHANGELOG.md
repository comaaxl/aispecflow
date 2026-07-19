# Changelog

All notable changes to this project will be documented in this file.

## [1.1.5] - 2026-07-19

### Changed
- `prune-review`: all three reviewer levels now share a **correctness kernel** - the difference between levels is scope, not methodology strength
- `prune-review`: added **cross-function contract** check to all three levels - internal calls are traced to their definitions and argument contracts (params/order/types/optional args) confirmed, not assumed. Catches bugs like a `%s`-placeholder SQL fed to an executor with no `params` argument
- `prune-review`: added **failure paths & side-effect consistency** check to all three levels, covering five categories: side-effect timing (audit/log/state written only on success), resource leaks on exceptions, partial-success rollback, error swallowing, and retry idempotency
- `prune-review`: task-level now matches change-level on single-task dimensions - added Production readiness axis and Consistency-with-existing-patterns axis (task-level still does NOT check cross-task or cannot-verify, by design)
- `prune-review`: change-level cross-task consistency now explicitly requires comparing same-kind operations across tasks (error handling, audit shape, auth, validation, return shape)
- `prune-review`: project-level Long-term drift renamed to Long-term drift & same-kind consistency, with explicit same-kind divergence flagging
- `prune-review`: all three reviewer levels may now read any source file under the project root to verify internal contracts and compare siblings; `{PROJECT_ROOT}` placeholder added to all templates and SKILL.md

## [1.1.4] - 2026-07-18

### Changed
- `prune-review`: Security promoted to a standalone axis in `task-reviewer.md` and `change-reviewer.md` (was buried as a bullet under Architecture; task-level had no Security check at all)
- `prune-review`: Important+ structural issues now require a named remedy (replace conditional chain with dispatcher / collapse duplicate branches / separate orchestration from business logic / move feature logic out of shared module / reuse canonical helper / make type boundary explicit / delete pass-through wrapper / extract helper or split file), so fixes stay surgical instead of vague signals
- `prune-review`: added a 4-level disagreement resolution hierarchy (technical facts > project style guide > software design principles > codebase consistency) to make pushback reasoned
- `prune-review`: added dead-code "report and ask, never auto-delete" sections to all three reviewer templates, aligned with `fix-agent.md`'s existing no-delete discipline
- `prune-review` (project-level): split the combined Dependencies & Security section into separate Dependencies (audit, lockfile, isolated upgrades, maintenance, transitive graph) and Security sections, plus a dedicated dead-code health-check section

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
