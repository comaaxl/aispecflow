---
name: prune-review
description: Dispatch a code reviewer subagent to evaluate completed work at three levels - task, change, or project. Use after implementation, at checkpoints, or for periodic health checks.
---

# Review - Code Review

Dispatch an independent code reviewer subagent to evaluate work at one of three
levels. The reviewer must be a separate agent with no access to your session
context - this prevents self-review bias.

**Core principle**: Review early, review often. Catch issues before they cascade.

## Three Levels

| Level | What it reviews | Range | When |
|---|---|---|---|
| **Change-level** (default) | All changes in an OpenSpec change | `base..HEAD` from the apply state file | After `/grow-apply` finishes; the usual review before archive |
| **Project-level** | The whole codebase | None - full read, no diff | Periodic health check; before release |
| **Task-level** | A single task's changes | `head..HEAD` from the apply state file | During `/grow-apply`, after a task's checkpoint commit |

## When to Use

**Change-level** (the usual one):
- After `/grow-apply` completes all tasks
- Before `/harvest-archive` (a soft reminder, not a hard gate)

**Project-level**:
- Periodically, as a codebase health check
- Before a release
- When you suspect long-term drift

**Task-level**:
- During `/grow-apply` (it triggers this automatically when enabled)
- Manually, to re-review a specific task

## Platform Adaptation

Different agent platforms have different support for key capabilities (e.g., dispatching subagents). Before running this skill, check whether your harness has special instructions.

If your harness appears here, read its reference file for platform-specific setup:

- Codex: `references/codex-tools.md`

## Process

### Step 0: Ask the user what level to review

When triggered manually, ask which level (default: change-level):

> "What level should I review?
> 1. **Change-level (recommended, default)** - Review all of this OpenSpec
>   change's changes. Range: base..HEAD.
> 2. **Project-level** - A health check on the whole codebase (full read, no diff).
> 3. **Task-level** - Review a single task's changes. (Usually triggered
>   automatically during `/grow-apply`; pick this only to re-review a specific
>   task manually.)"

When triggered by `/grow-apply` at a checkpoint, the level is already
**task-level** - skip this question.

**No-git handling**: if there is no git, task-level is unavailable (explain why -
no commit boundaries); change-level degrades to the whole working tree
(imprecise); project-level is unaffected.

### Step 1: Derive the review range internally

Compute the range from the apply state file (`openspec/changes/<change>/.aispecflow-apply-state.md`, see `grow-apply/references/apply-state.md`). Never ask the user for git commands.

- **Task-level**: read `head` from the state file; range = `head..HEAD`. If
  `head` is unset (no checkpoint yet), tell the user to commit a checkpoint
  first or explain there is no task boundary.
- **Change-level**: read `base` from the state file; range = `base..HEAD`. If
  `base: no-git`, review the whole working tree instead.
- **Project-level**: no range. The reviewer reads the whole codebase.

If the state file is missing or `base` cannot be derived (and it is not
`no-git`): ask the user to provide a base SHA, or fall back to the whole working
tree.

### Step 2: Prepare context

Collect:
- **Description**: What was built (from `proposal.md` or ask user)
- **Requirements**: Completed tasks from `tasks.md` + `specs/` (not needed for
  project-level)
- **Change name**: The OpenSpec change name, if applicable

### Step 3: Prepare the diff as a file and dispatch the reviewer

**CRITICAL**: The reviewer must be a separate subagent, not you acting as
reviewer. It gets ONLY its template and file paths - never your session context.

For **task-level** and **change-level**, write the diff to a uniquely named file
so the reviewer reads one file instead of running git commands itself (this keeps
the reviewer's context clean and the diff deterministic):

```bash
DIFF_FILE=$(mktemp /tmp/aispecflow-review-XXXXXX.md)
{
  echo "## Commits"
  git log --oneline <base>..<head>
  echo
  echo "## Stat"
  git diff --stat <base>..<head>
  echo
  echo "## Diff"
  git diff -U10 <base>..<head>
} > "$DIFF_FILE"
echo "Diff written to: $DIFF_FILE"
```

For **project-level**, do not generate a diff. Give the reviewer the project root
path and the `openspec/specs/` path; it reads the whole codebase.

Select the template by level and fill placeholders:
- **Task-level**: `references/task-reviewer.md` - placeholders `{DIFF_FILE}`,
  `{TASK_LINE}` (the tasks.md line), `{SPEC_FILES}`.
- **Change-level**: `references/change-reviewer.md` - placeholders
  `{DIFF_FILE}`, `{PROPOSAL}`, `{SPECS_DIR}`, `{TASKS_FILE}`.
- **Project-level**: `references/project-reviewer.md` - placeholders
  `{PROJECT_ROOT}`, `{SPECS_DIR}`.

The reviewer checks (level-dependent - see each template):
1. **Plan/spec alignment**: Implementation matches requirements? Deviations justified?
2. **Code quality**: Separation of concerns, error handling, type safety, DRY
3. **Architecture**: Sound design, scalability, security, integration
4. **Testing**: Tests verify behavior (not mocks), edge cases covered, all passing
5. **Production readiness**: Migration strategy, backward compatibility, docs
6. **Cannot-verify-from-diff** (change-level): items whose evidence lives in
   unchanged code or spans tasks - flagged with ⚠️ for you to resolve

### Step 4: Evaluate the review

When the reviewer returns:

```
Reviewer output:
  Strengths: ...
  Issues: Critical (N), Important (N), Minor (N)
  Assessment: Ready to merge? [Yes | No | With fixes]
  Cannot-verify-from-diff (if change-level): ⚠️ items
```

**For each issue, verify before acting:**
- Is it technically correct for THIS codebase?
- Does it break existing functionality?
- Is there a reason for the current implementation?

**For each ⚠️ cannot-verify item** (change-level): you hold the cross-task
context the reviewer lacks. Confirm whether it is a real gap; if so, treat it as
a failed spec review.

**If reviewer is wrong**: Push back with technical reasoning.
**If reviewer is correct**: Fix Critical immediately, Important before archive, Minor note for later.

### Step 5: Act on feedback

```
Critical -> Fix now, re-review
Important -> Fix before archive
Minor -> Note for future
```

**How fixes are done** depends on the `fix_mode` in the apply state file (chosen
once at `/grow-apply` start):
- `inline` - fix in this conversation
- `subagent` - dispatch a fix agent (see `references/fix-agent.md`)

After fixing, make a new commit (task-level fixes use `fix(review-task-X): ...`).
**Only append commits - never amend or squash.**

**Control returns to the user after fixing.** Re-review is optional and decided
by the user - do not auto-loop review.

After all Critical + Important resolved:

> "Code review complete. All Critical and Important issues resolved. Ready for
> `/harvest-archive`."

## Receiving Review Feedback (Discipline)

```
1. READ: Complete feedback without reacting
2. UNDERSTAND: Restate requirement in own words
3. VERIFY: Check against codebase reality
4. EVALUATE: Technically sound for THIS codebase?
5. RESPOND: Technical acknowledgment or reasoned pushback
6. IMPLEMENT: One item at a time, test each
```

**Never:**
- Performative agreement ("You're absolutely right!")
- Blind implementation (verify first)
- Argue with valid technical feedback

**Push back when:**
- Suggestion breaks existing functionality
- Reviewer lacks full context
- Violates YAGNI
- Technically incorrect for this stack

## Output

```
## Review Complete

**Change:** <name>
**Level:** task / change / project
**Reviewer Verdict:** Approved / Approved with fixes / Needs work
**Scope reviewed:** <range or "whole codebase">

### Resolved
- [x] Critical: <issue> -> Fixed in <file>
- [x] Important: <issue> -> Fixed in <file>

### Deferred (Minor)
- [ ] <issue> - noted for future

Prompt the user: "Review complete. Run `/harvest-archive` to archive this change."
```

## Guardrails

- **Must use a separate subagent, not self-review.** Reviewer is always independent and read-only.
- **Ask the level, not git commands.** Derive the range internally from the state file.
- **Default level is change-level** when triggered manually.
- **Task-level is unavailable without git** (no commit boundaries).
- **Reviewer input goes through file paths**, never pasted into main conversation.
- **Review is always a subagent - there is no "inline review" option.**
- **Handle missing git.** If git is not available, change-level reviews the working tree directly; task-level is skipped.
- **Never skip review** when it has been requested.
- **Verify before implementing feedback.**
- **No performative agreement.**
- **After fixing, re-review is the user's call - never auto-loop.**
- **Only append commits - never amend or squash by default.**
- **Communicate in the user's language.** Match the language the user writes in throughout the session. Never mix languages in a single response.
