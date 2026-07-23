---
name: axl-dev-flow
description: Orchestrate the complete development lifecycle — repo discovery, requirements grill, spec generation, TDD implementation, code review, archive, and docs refresh. Use when starting a new feature or change from scratch.
---

# Axl Dev Flow — Full Development Lifecycle

Orchestrate the complete development lifecycle through 7 phases:

```
0. DISCOVER  → Understand the project (optional if project is already known)
1. GRILL     → Clarify requirements
2. SPEC      → Generate formal SDD artifacts
3. APPLY     → Implement with TDD
4. REVIEW    → Code review
5. ARCHIVE   → Archive the change
6. REFRESH   → Update project documentation
```

**Core principle**: Each phase is gated — you cannot proceed until the completion criterion is met. You can enter at any phase if prerequisites are satisfied.

## Phase 0: Discover (Optional)

**When to run**: Starting work on an unfamiliar project, or if `docs/project-overview.md` doesn't exist or may be stale.

**What happens**: Run `/terrain-scan` to scan the codebase and produce `docs/project-overview.md`.

**Completion criterion**: `docs/project-overview.md` exists and user confirms it's accurate.

**Skip if**: `docs/project-overview.md` exists AND you've verified it matches the current codebase (spot-check key claims). If in doubt, re-run discovery.

> "Phase 0: Let's discover this project first. Run `/terrain-scan`?"

## Phase 1: Grill

**When to run**: Always before starting a new feature or change.

**What happens**: Run `/seed-grill` to relentlessly interview until what the user wants is unambiguous, then produce the document(s) they asked for. Output is intent-driven - could be `requirements.md`, `prd.md`, `tech-architecture.md`, `api-design.md`, UI prototype, or any combination. Also updates `CONTEXT.md` and creates ADRs.

**Completion criterion**: User confirms the produced document(s) capture everything correctly.

**Prerequisites**: `docs/project-overview.md` is helpful but not required.

> "Phase 1: Let's grill the requirements. Run `/seed-grill`?"

## Phase 2: Spec

**When to run**: After requirements are confirmed.

**What happens**: Run `/bloom-spec` to generate OpenSpec proposal, specs, design, and tasks (with TDD instructions embedded).

**Completion criterion**: All OpenSpec artifacts created and `openspec status` shows all `applyRequires` as done.

**Prerequisites**: `docs/requirements.md` (if missing, spec will ask for description). Note: grill may produce other documents instead of or in addition to requirements.md - spec discovers all docs in `docs/` and lets the user select which to use as context.

> "Phase 2: Let's turn requirements into specs. Run `/bloom-spec`?"

## Phase 3: Apply

**When to run**: After specs and tasks are ready.

**What happens**: Run `/grow-apply` to implement all tasks using TDD (RED -> GREEN -> REFACTOR per task), with a git state check, checkpoint commits per task, and optional task-level review.

**Completion criterion**: All tasks checked, all tests green, no skipped tests, apply state file recorded (base + head). If the user opted into the optional project checks (lint/type/integration, config-driven), those pass or the user explicitly proceeded.

**Prerequisites**: `tasks.md` exists with checkbox tasks.

> "Phase 3: Let's implement with TDD. Run `/grow-apply`?"

## Phase 4: Review

**When to run**: After all tasks are implemented.

**What happens**: Run `/prune-review` to dispatch a code reviewer subagent at the change level (default) or project level. Fix Critical and Important issues.

**Completion criterion**: Reviewer verdict is "Approved" or "With fixes" and all Critical + Important issues resolved.

**Prerequisites**: All tasks complete, code committed (grow-apply checkpoint commits or user-committed).

> "Phase 4: Let's review the code. Run `/prune-review`?"

## Phase 5: Archive

**When to run**: After review is approved.

**What happens**: Run `/harvest-archive` to sync delta specs and move the change to the archive.

**Completion criterion**: Change moved to `openspec/changes/archive/`.

**Prerequisites**: All tasks done, review approved.

> "Phase 5: Let's archive this change. Run `/harvest-archive`?"

## Phase 6: Refresh Docs
**When to run**: Anytime docs are stale — after archiving, or whenever you notice gaps.
**When to run**: After archiving.

**What happens**: Run `/renew-docs` to update README and `docs/project-overview.md` to reflect the current codebase.
**Prerequisites**: None (usable anytime). When run after `/harvest-archive`, the archived change provides context.
**Completion criterion**: Documentation updated and synced with current state.


> "Phase 6: Let's refresh the project docs. Run `/renew-docs`?"

## Entry Points

You don't have to start at Phase 0. Valid entry points:

| If you have... | Start at |
|---------------|----------|
| Nothing, new project | Phase 0 (discover) |
| Known project, new idea | Phase 1 (grill) |
| Clear requirements document | Phase 2 (spec) |
| Existing OpenSpec change, ready to code | Phase 3 (apply) |
| Code written, needs review | Phase 4 (review) |
| Reviewed, ready to ship | Phase 5 (archive) |
| Documentation is stale | Phase 6 (refresh) |

## Quick-Start

User says "let's build X" → Check what exists:

1. Is `docs/project-overview.md` present AND verified current? → Yes → skip Phase 0. No → Phase 0
2. `docs/requirements.md` for X? → No → Phase 1
3. OpenSpec change for X? → No → Phase 2
4. Otherwise → Start at the earliest missing phase

## Orchestration Rules

1. **Announce each phase transition.** "Phase N: <name>" with a brief description.
2. **Verify completion criteria.** Don't proceed without confirmation.
3. **Don't skip phases without asking.** "Phase 1 (grill) is normally needed. Skip it?"

## Reference

Detailed phase instructions are in `references/`:
- `references/phase-0-discover.md`
- `references/phase-1-grill.md`
- `references/phase-2-spec.md`
- `references/phase-3-apply.md`
- `references/phase-4-review.md`
- `references/phase-5-archive.md`
- `references/phase-6-refresh.md`
## Guardrails

- **Communicate in the user's language.** Match the language the user writes in throughout the session. Never mix languages in a single response.
