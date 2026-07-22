# Changelog

All notable changes to this project will be documented in this file.

## [1.1.8] - 2026-07-22

### Added
- `seed-grill`: new "Commit artifacts (if git)" step at the end. After the user confirms requirements.md, commits the files this skill produced (docs/requirements.md, CONTEXT.md, docs/adr/*) so the working tree is clean for /bloom-spec. Only commits this skill's own files (no `git add -A`), confirms scope with the user first, append-only. Skips silently if no git or user declines.
- `bloom-spec`: new "Step 5: Commit artifacts (if git)". Commits proposal.md / design.md / tasks.md / specs/ so /grow-apply starts on a clean tree. Same discipline: only this skill's files, user confirms scope, append-only, skips silently.
- `grow-apply`: Step 5 now probes **prerequisites** for checks that need them (integration tests that skip unless an env var is set, feature-gated tests, etc.). Scans test config files (NOT only conftest.py - also pytest.ini, pyproject.toml, jest.config, Cargo.toml, Go test files, marker descriptions) for prerequisite patterns, checks whether each is currently met, and marks unmet ones explicitly in the offer ("需设 {ENV_VAR}，当前未设 -> 将被跳过").
- `grow-apply/references/project-checks-probe.md`: new "Prerequisites (env vars / feature gates)" section with a pattern table (Python/JS/Go/Rust) and false-green reporting rules.

### Changed
- `grow-apply`: Step 5 result reporting no longer treats "exit code 0" as "all passed" when checks were skipped due to unmet prerequisites. A skipped integration test is reported as a false-green warning, not a pass - the most dangerous outcome is the user thinking integration is verified when it was never run.

## [1.1.7] - 2026-07-22

### Changed
- `renew-docs`: removed hardcoded `README.md` + `docs/project-overview.md` assumption. Now dynamically probes the project's documentation inventory (README, rule files like CLAUDE.md/AGENTS.md, docs/, ADRs, changelog, deploy/ops docs, API docs, schema/migration docs, project-specific docs) and refreshes whatever actually exists. No docs found -> stops and tells the user instead of inventing documents.
- `renew-docs`: Step 3 now checks each document by its **type** (README-type / architecture-type / rule-file-type / changelog-type / deploy-ops-type / API-type / schema-migration-type) instead of a fixed two-document checklist. Each type has its own verification questions.
- `renew-docs`: added a **knowledge placement guide** (drawn from neat-freak best practices) deciding where an update belongs - rule files vs README/docs vs changelog vs ADR - to avoid duplicating content across documents or putting it in the wrong place.
- `renew-docs`: Step 4/6 examples generalized to show multiple document types, with a note that the actual list depends on what was probed.

### Added
- `renew-docs`: new Step 2.5 (probe documentation inventory) with a probe-signal table that is explicitly a starting point, not exhaustive.

## [1.1.6] - 2026-07-22

### Added
- `grow-apply`: new **Step 5 - project checks (optional, user-driven)** after all tasks complete. Probes the project's own configured static checks and integration tests (lint, type check, integration tests, format check, dependency security scan, coverage gates), then offers to run them (all / user-selected subset / skip). Runs only against changed files when the tool supports it; the tool's rule set and ignores still apply from the project config. Reports results and suggests `/prune-review`; never auto-runs it. No config found -> skips silently.
- `grow-apply`: new reference file `references/project-checks-probe.md` - the config-signal table mapping project files (pyproject.toml, package.json, tsconfig.json, golangci.yml, Cargo.toml, etc.) to check categories. Explicitly a starting point, not exhaustive.

### Changed
- `axl-dev-flow` Phase 3 completion criterion: if the user opted into project checks, those pass or the user explicitly proceeded.
- `axl-dev-flow` Phase 3 reference (`phase-3-apply.md`): documented the new Step 7 project-checks step and updated the completion criterion.
- `README.md` / `README-zh.md`: grow-apply description now mentions the optional config-driven project checks.

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
