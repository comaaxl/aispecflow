---
name: grow-apply
description: Implement tasks from an OpenSpec change using test-driven development. Every task follows RED-GREEN-REFACTOR. Use when ready to implement a spec'd change.
---

# Apply - TDD Implementation

Implement tasks from an OpenSpec change. **Outer loop**: iterate through tasks.md. **Inner loop**: RED -> GREEN -> REFACTOR for every production code change.

## Iron Law

```
NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST.
```

If you wrote code before a test, delete the code. Start over. No exceptions.

## Pre-flight

### Step 0: Git state check

Before anything else, detect git availability and working-tree cleanliness. This
determines whether review ranges can be computed and whether task-level review is
available. See `references/apply-state.md` for the state file format.

**Has git + clean tree** - record BASE and write the state file:
```bash
BASE=$(git rev-parse HEAD)
```
Write `openspec/changes/<change>/.aispecflow-apply-state.md` with `base`, today's
date as `started`, and `per_task_review: on` (default).

**Has git + dirty tree** - STOP and ask the user. Do not proceed on a dirty tree
without an explicit choice - uncommitted changes pollute review diff ranges,
mixing unrelated changes into "this change".

> "The working tree has uncommitted changes (<list files>). How do you want to
> handle them before we start?
> 1. **Commit them now** - I'll write the commit message, you review it
> 2. **Stash them** - set them aside, restore after this change
> 3. **Abort** - I'll handle them myself and come back"

**No git** - STOP and ask the user:

> "This project has no git. What do you want to do?
> 1. **`git init` first (recommended)** - I run `git init`, commit the current
>   files as the initial commit, then start over with full review support (base
>   recording, task-level review, rollback, recovery).
> 2. **Continue without git** - Proceed, but review is degraded:
>   - Cannot record a change base; change-level review degrades to the whole
>     working tree (imprecise)
>   - Task-level review unavailable (no task commit boundaries)
>   - No rollback; progress recovery relies only on tasks.md checkboxes"

**If the user picks `git init`**: run `git init`, stage and commit the current
files as an initial commit (`git add -A && git commit -m "Initial commit"`),
then treat this as the "clean tree" case above - record `base` from that commit
and continue with full review support. Do NOT write `base: no-git`.

**If the user picks continue without git**: write the state file with
`base: no-git` and force `per_task_review: off`.

### Step 1: Select the change

If a name is provided, use it. Otherwise:
- Run `openspec list --json` to get available changes
- If only one active change exists, auto-select it
- If multiple, ask the user to choose

Announce: "Implementing change: `<name>`"

### Step 2: Load context

```bash
openspec status --change "<name>" --json
openspec instructions apply --change "<name>" --json
```

Read all context files listed in the apply instructions output.

### Step 3: Choose implementation mode

Ask the user three questions, **one at a time**. Each option must explain what
happens if chosen - no jargon without explanation. Recommend a default for each.

**Question 1 - How to write the code:**

> "How would you like to implement these tasks?
> 1. **Test-driven (recommended)** - For each task, write a failing test first,
>    then the minimal code to pass it, then refactor. Every step has a test
>    backing it.
> 2. **Straight implementation** - Make code changes directly, add tests after
>    if needed. Faster, but you lose the step-by-step test safety net."

If **TDD**: follow "The TDD Implementation Loop" below.
If **Straight**: for each task, make code changes, verify they work, mark
complete, move on. Skip the RED-GREEN-REFACTOR inner loop.

**Question 2 - Task-level review during implementation:**

First explain: review is done by an independent, read-only reviewer (it cannot
see this conversation - it only reads code and task notes) that catches
implementation-vs-requirement gaps, over-engineering, missing tests. Regardless
of this choice, you can do a full change-level review after all tasks are done.
This question is whether to also review after each task.

> "Do task-level review during implementation?
> 1. **Yes (recommended, default)** - Review after each task is done. Catches
>    problems early; useful when there are many tasks or you worry about drift.
>    Cost: one extra review round per task.
> 2. **No** - Only review once after everything is done. Saves effort when tasks
>    are few and changes are direct."

When there is no git: skip this question, force "No", and explain why (no task
commit boundaries - the reviewer cannot scope a single task's changes).

**Question 3 - How to fix review findings** (only if Question 2 is "Yes"; asked
**once** and recorded for the whole apply session):

First explain: the reviewer only flags problems, it does not change code. Once
problems are flagged, someone has to fix them. This choice is recorded and
reused for every task - not re-asked.

> "How would you like review findings fixed?
> 1. **I fix them in this conversation (recommended, when issues are few)** -
>    The reviewer lists the problems; I verify each and fix them here, then
>    recommit. Simple and direct.
> 2. **Dispatch a separate fix agent (when there are many issues)** - If a
>    review surfaces a lot of problems, I spawn a fix-only assistant to handle
>    them in batch so this conversation doesn't get too long. It reports back
>    when done."

Both options recommit (new commits only, never rewrite history). After fixing,
control returns here so **you** decide whether to re-review (not automatic).

Write all three answers to the state file: `mode`, `per_task_review`, `fix_mode`.

Note: **Review is always done by an independent read-only subagent - it is not
a choice.** Question 3 only asks how fixes are done, not how review is done.

### Step 4: Show current state

Display:
- Schema being used
- Tasks progress: "N/M tasks complete"
- Remaining tasks overview

## The TDD Implementation Loop

### Outer Loop: Iterate tasks

For each pending task in `tasks.md`:

```
┌─────────────────────────────────────────────┐
│  Task X.Y: <description>                    │
│                                             │
│  ┌─ Inner Loop: TDD ─────────────────────┐  │
│  │  RED:    write failing test            │  │
│  │  VERIFY: watch it fail correctly       │  │
│  │  GREEN:  write minimal code to pass    │  │
│  │  VERIFY: watch test + all others pass  │  │
│  │  REFACTOR: clean up, keep green        │  │
│  │  REPEAT: next test for this task       │  │
│  └────────────────────────────────────────┘  │
│                                             │
│  ✓ Mark task complete: `- [x] X.Y ...`     │
│  ↻ Checkpoint commit + optional review     │
└─────────────────────────────────────────────┘
```

### Inner Loop: TDD Discipline

**RED - Write a failing test**
- One test, one behavior
- Clear name describing what it verifies
- Use real code, mock only at system boundaries (external APIs, databases, time)
- The test must verify observable behavior through public interfaces

**VERIFY RED - Watch it fail**
- Run the specific test: confirm it FAILS
- Confirm failure is because the feature is missing, not a typo
- If it passes, the test is wrong - fix the test
- **Never skip this step**

**GREEN - Write minimal code**
- Only enough code to make THIS test pass
- Don't add features, abstractions, or "future-proofing"
- YAGNI: no speculative parameters, branches, or flexibility the test doesn't require
- Don't refactor other code

**VERIFY GREEN - Watch it pass**
- Run the specific test: confirm it PASSES
- Run the full test suite: confirm no regressions
- Output must be clean - no errors, no warnings

**REFACTOR - Clean up**
- Only after all tests pass
- Extract duplication, improve names, extract helpers
- DRY: eliminate repeated logic, but stop at extraction the tests justify - don't abstract speculatively
- Keep tests green after each refactor step
- Don't add behavior during refactoring

**REPEAT** - Next test for the same task until the task is complete.

### Task Completion

After all tests for a task pass:

1. **Mark the task complete** in `tasks.md`: replace the specific `- [ ] X.Y ...`
   line with `- [x] X.Y ...`. **One line at a time. Never use global regex or
   batch sed across the entire file.**

2. **Checkpoint commit** (skip if no git):
   ```bash
   git add -A
   git commit -m "task(X.Y): <description>"
   ```
   Update the state file's `head` field to this commit's SHA. This checkpoint is
   what task-level review scopes against (`head..HEAD`).

3. **Task-level review** (only if `per_task_review: on`; every task triggers it):
   Hand control to `/prune-review` at the **task level**. State:
   > "Triggering task-level review for task X.Y. Handing control to
   > `/prune-review` (task-level)."

   - The review is always performed by an independent, read-only reviewer
     subagent - it cannot see this conversation. It only reads the task notes,
     the diff for this task, and the relevant spec files. **grow-apply itself
     never acts as the reviewer.**
   - When the reviewer returns issues, come back here and verify each one
     (read -> understand -> check against code reality -> evaluate -> respond ->
     fix). Distinguish real fixes from pushback (push back with technical
     reasoning when the reviewer is wrong).
   - Fix according to `fix_mode` in the state file (chosen once at start, reused
     for every task):
     - `inline` - fix here in this conversation
     - `subagent` - dispatch a fix agent (see `prune-review/references/fix-agent.md`)
   - After fixing, make a new commit:
     ```bash
     git commit -m "fix(review-task-X): <description>"
     ```
   - **Control returns here after fixing.** You (the user) decide whether to
     re-review (trigger `/prune-review` task-level again) - it is **not
     automatic**. This avoids review-fix-review loops and subagent overuse.
   - Review logic lives in `/prune-review`. grow-apply only triggers it and
     handles the fixes.

4. Show progress: "Task X.Y complete (N/M done)"

5. Move to next task.

### Pause Conditions

Stop and ask if:
- A task is unclear or ambiguous
- Implementation reveals a design issue (suggest updating artifacts)
- An error or blocker occurs that can't be resolved
- User interrupts

## Straight Implementation Mode

When the user chooses straight implementation:

For each pending task:
- Make the code changes required
- Keep changes minimal and focused
- YAGNI: only what the task requires - no speculative features or premature abstraction
- Verify the change works
- Mark task complete: replace the specific `- [ ] X.Y ...` line with `- [x] X.Y ...` - **one line at a time, no batch regex**
- **Checkpoint commit + optional task-level review** - same as the TDD outer loop
  (steps 2-3 above), just without the RED-GREEN-REFACTOR inner loop
- Continue to next task

## What to Test

**Good tests** (write these):
```python
# Tests observable behavior through public interface
def test_user_can_authenticate_with_valid_credentials():
    result = authenticate("user@example.com", "correct-password")
    assert result.success is True
    assert result.token is not None

# Clear name, tests real behavior, one logical assertion
def test_authenticate_rejects_expired_token():
    expired_token = create_token(expires_in=-3600)
    result = validate_token(expired_token)
    assert result.valid is False
```

**Bad tests** (avoid these):
```python
# Tests implementation details - mocking internal collaborators
def test_authenticate_calls_password_hasher():
    mock_hasher = Mock(PasswordHasher)
    authenticate("user", "pass")
    assert mock_hasher.verify.called  # BAD: testing HOW, not WHAT
```

## When to Mock

Mock at **system boundaries only**:
- External APIs (payment, email, third-party services)
- Databases (prefer test database when possible)
- Time/randomness
- File system (when practical)

**Never mock** your own classes, modules, or internal collaborators.

## Completion

```
## Implementation Complete

**Change:** <name>
**Progress:** N/N tasks complete ✓

### Completed
- [x] Task 1: ...
- [x] Task 2: ...
...

All tasks complete. You can now run `/prune-review` to do a
**change-level review** (all of this change's changes, range = base..HEAD) or a
**project-level review** (a health check on the whole codebase).
```

If there is no git, note that change-level review degrades to the whole working
tree (imprecise); project-level review is unaffected.

## Guardrails

- **Delete code written before tests.** Iron Law. No exceptions (in TDD mode).
- **One test at a time.** Don't write multiple tests before implementing (in TDD mode).
- **Verify RED every time.** Never skip verification (in TDD mode).
- **Mark tasks one at a time.** Replace the exact line, never batch regex across the whole file.
- **Dirty working tree must be asked about.** Never proceed without an explicit user choice.
- **No-git must be explained and asked about.** Never silently continue.
- **Review is always an independent read-only subagent.** grow-apply only triggers and fixes; it never acts as reviewer.
- **Only append commits, never rewrite history.** No amend, no default squash.
- **A checkpoint commit must precede reviewable work** (skipped under no-git degradation).
- **Ask the three questions one at a time.** Each option explains what happens if chosen - no bare jargon. Defaults: test-driven, task-level review on, fix in conversation.
- **Fix mode is asked once** (at start) and reused for every task - never re-asked per task.
- **After fixing, control returns to the user** to decide on re-review - never auto-loop review.
- **Pause on ambiguity.** Don't guess - ask.
- **Communicate in the user's language.** Match the language the user writes in throughout the session. Never mix languages in a single response.
