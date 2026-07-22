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

**Resume check (do this first, before recording a new BASE).** If a state file
already exists for this change, a previous `/grow-apply` may not have finished.
Do NOT blindly overwrite it. Cross-check four sources:

1. The state file's `base`, `head`, and `last_completed_task` fields.
2. `tasks.md` checkboxes - how many tasks are marked `[x]`.
3. `git log <base>..HEAD --oneline` - how many `task(...)` / `fix(review-task-...)`
   commits actually exist.
4. Cross-validate `last_completed_task` against tasks.md: if the state file says
   task X is the last completed but tasks.md does not mark X as `[x]` (or vice
   versa), the state is inconsistent.

If they are consistent and all tasks are complete, the previous apply finished
normally - proceed to start fresh (you may overwrite the state file). If they are
inconsistent OR there are incomplete tasks, the previous apply was interrupted.
STOP and ask the user:

> "I found an existing apply state for this change - the previous `/grow-apply`
> may not have finished. Here's what I see:
> - tasks.md: <N> of <M> tasks marked complete
> - last completed task (state file): <X.Y or none>
> - git log since base: <list of task/fix commits>
> - last checkpoint (head): <sha>
>
> A committed-but-unmarked task (git has `task(X.Y)` but tasks.md doesn't mark
> X.Y `[x]`) means its review was unfinished. In-progress task state
> (half-written code, mid-self-review fixes) is NOT recoverable. How do you want
> to proceed?
> 1. **Continue from the first incomplete task** - I resume at the first task not
>   marked `[x]`. Note: the interrupted task's in-progress state is lost; I'll
>   re-implement it from scratch (its half-finished changes, if any, should be
>   checked first).
> 2. **Re-review the last completed task first** - I run `/prune-review` on the
>   most recent checkpoint before continuing, in case it was marked done with
>   unresolved issues.
> 3. **Abort** - I'll inspect the state myself and come back."

If the user picks 1: before re-implementing the interrupted task, check the
working tree and `git status` for half-finished changes from that task; offer to
commit or discard them (do not silently carry them into the re-implementation).
If the user picks 2: run the review, resolve findings, then continue from the
first incomplete task.

If no state file exists, this is a fresh apply - proceed.

**Has git + clean tree** - record BASE and write the state file:
```bash
BASE=$(git rev-parse HEAD)
```
Write `openspec/changes/<change>/.aispecflow-apply-state.md` with `base`, today's
date as `started`. `per_task_review` and `fix_mode` are filled in after Step 3's
three questions (default `per_task_review: final`).

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
`base: no-git` and force `per_task_review` to `final` or `none` (`every` needs
commit boundaries).

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

**Question 2 - Subagent review depth:**

First explain: after each task's tests pass, the main conversation always does a
quick **self-review** (a checklist pass - see `references/self-review-checklist.md`)
and fixes what it finds, inline. That is the baseline, always on, no subagent.

This question is whether to ALSO dispatch an independent read-only subagent
reviewer (it cannot see this conversation - it reads code and task notes, and
catches blind spots self-review misses), and at what granularity:

> "Beyond the always-on self-review, do you want an independent subagent review?
> 1. **After every task** - self-review, then a subagent reviews each task.
>    Catches blind spots early. Cost: one subagent round per task (slow).
> 2. **Once at the end (recommended, default)** - only self-review per task;
>    after all tasks are done, one subagent reviews the whole change. Fast
>    during implementation, with an independent backstop at the end.
> 3. **No subagent review** - only self-review. Fastest. For few tasks, direct
>    changes, when you're confident."

- Store the choice as `per_task_review`: `every` / `final` / `none`.
- When there is no git: option 1 is unavailable (no task commit boundaries for
  the subagent to scope). Force option 2 or 3, and explain why.

**Question 3 - How to fix subagent review findings** (asked when Question 2 is
`every` or `final` - i.e. a subagent WILL be dispatched; skipped when `none`;
asked **once** and recorded for the whole apply session):

First explain: the subagent reviewer only flags problems, it does not change
code. Once problems are flagged, someone has to fix them. This choice is
recorded and reused for every subagent review (per-task or final) - not
re-asked. (Self-review findings are always fixed inline - this choice does not
apply to them.)

> "How would you like subagent review findings fixed?
> 1. **I fix them in this conversation (recommended, when issues are few)** -
>    The reviewer lists the problems; I verify each and fix them here, then
>    recommit. Simple and direct.
> 2. **Dispatch a separate fix agent (when there are many issues)** - If a
>    review surfaces a lot of problems, I spawn a fix-only assistant to handle
>    them in batch so this conversation doesn't get too long. It reports back
>    when done."

Both options recommit (new commits only, never rewrite history). After fixing,
control returns here so **you** decide whether to re-review (not automatic).

Write all three answers to the state file: `mode`, `per_task_review`, `fix_mode`
(`fix_mode` is unset when `per_task_review: none`).

Note: **Subagent review (when enabled) is always an independent read-only
subagent - that is not a choice.** Question 3 only asks how fixes are done, not
how review is done. Self-review is always the main conversation, always inline.

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
│  🔍 Self-review (always) + fix + re-test   │
│  ↻ Checkpoint commit                        │
│  ✓ Optional subagent review (per_task_review)│
│  ✓ Mark task complete: `- [x] X.Y ...`     │
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

1. **Self-review** (always, every task - this is the baseline). Run through the
   checklist in `references/self-review-checklist.md` (spec compliance, obvious
   gaps, test quality, naming, cut corners). Fix anything you find **inline**
   (self-review fixes are always inline - `fix_mode` does not apply to them).
   After fixing, **re-run the tests** and confirm they still pass. Do not
   commit until green.

2. **Checkpoint commit** (skip if no git). Capture the pre-task commit, then
   commit:
   ```bash
   TASK_BASE=$(git rev-parse HEAD)   # the commit before this task's changes
   git add -A
   git commit -m "task(X.Y): <description>"
   ```
   The review range is `TASK_BASE..HEAD` (covers this task's full changes,
   including the self-review fixes). Update the state file's `head` field to
   the new commit's SHA. Do **not** advance `last_completed_task` yet - that
   happens only when the task is marked `[x]` (step 4), so a committed-but-
   unmarked task is detectable as "review unfinished" on resume.

3. **Subagent review** (only if `per_task_review: every`; skipped for `final`
   and `none`): hand control to `/prune-review` at the **task level**. State:
   > "Triggering task-level review for task X.Y. Handing control to
   > `/prune-review` (task-level)."

   - The review is always performed by an independent, read-only reviewer
     subagent - it cannot see this conversation. It only reads the task notes,
     the diff for this task (`TASK_BASE..HEAD`), and the relevant spec files.
     **grow-apply itself never acts as the reviewer.**
   - When the reviewer returns issues, come back here and verify each one
     (read -> understand -> check against code reality -> evaluate -> respond ->
     fix). Distinguish real fixes from pushback (push back with technical
     reasoning when the reviewer is wrong).
   - Fix according to `fix_mode` in the state file (chosen once at start, reused
     for every subagent review - per-task or final):
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

4. **Mark the task complete** in `tasks.md` - only AFTER self-review (and, when
   applicable, subagent review findings) are resolved. Replace the specific
   `- [ ] X.Y ...` line with `- [x] X.Y ...`. **One line at a time. Never use
   global regex or batch sed across the entire file.** A task is not "complete"
   while it still has unresolved review issues. **Now** advance the state
   file's `last_completed_task` to this task's number (e.g. `1.3`).

5. Show progress: "Task X.Y complete (N/M done)"

6. Move to next task.

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
- **Self-review -> checkpoint commit -> optional subagent review -> mark complete** -
  same order as the TDD outer loop (steps 1-4 above), just without the
  RED-GREEN-REFACTOR inner loop. Self-review (checklist) and its inline fixes
  happen before the checkpoint commit; subagent review runs only if
  `per_task_review: every`; mark `[x]` only after review findings are resolved,
  then advance `last_completed_task`.
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

All tasks complete.
```

### Step 5: Project checks (ask first, never auto-run)

After all tasks are complete, **probe the project's configured checks** by
looking for the project's own configuration files and mapping them to check
categories. See `references/project-checks-probe.md` for the signal table (a
starting point, NOT exhaustive - extend it to match what you actually find).

**MUST ask the user before running anything.** Present what you probed and the
run options, then STOP and wait for the user's answer. Do NOT run any check
until the user explicitly picks an option. The user needs this prompt to
provide prerequisites (env vars, DB connections, etc.) that some checks need -
running without asking robs them of that chance.

**Probe prerequisites for checks that need them.** Some checks only run when an
external resource is available - e.g. integration tests that skip unless an env
var is set, or a feature flag is on. Scan the project's **test config files**
(not only `conftest.py` - also `pytest.ini`, `pyproject.toml [tool.pytest]`,
`jest.config.*`, `package.json`, `Cargo.toml`, `go.mod`-adjacent test files,
etc.) for prerequisite patterns:
- `os.environ.get("XXX")` / `os.Getenv("XXX")` followed by `skip` (Python/Go)
- `process.env.XXX` followed by `describe.skip`/`it.skip` (JS/TS)
- feature gates / build tags that gate integration tests (Rust `#[cfg(feature=...)]`, Go `//go:build`)
- marker descriptions mentioning env vars / prerequisites (e.g. pyproject
  `markers = ["integration: 需设 XXX 环境变量"]`)

For each prerequisite found, **check whether it is currently satisfied** (is the
env var set? is the feature on?). This is a starting point, NOT exhaustive -
extend the patterns to match what you actually find.

**If checks were probed, ask the user** (replace placeholders with probed
results; show only the rows that matched; for checks with unmet prerequisites,
mark them explicitly; ask in the user's language - the Chinese below is
reference wording):

> "所有 task 已完成。是否按项目配置做校验？
>
> 检测到项目已配置以下校验：
> - {类别1}: {工具} ({配置位置})
> - {类别2}: {工具} ({配置位置})
> - 集成测试: {工具} (需设 {ENV_VAR}，当前未设 -> 将被跳过)
> - ...
>
> 注意：部分校验有前置条件未满足（如上标注）。如需运行这些校验，请先提供所需环境变量或配置，再选择。
>
> 请选择：
> 1. 全部跑（推荐）- 跑上述所有校验（注：未满足前置条件的会被跳过；如需运行请先配置）
> 2. 自选 - 你指定跑哪些（从上述列表里选子集）
> 3. 跳过 - 不跑任何校验，直接进入下一步"
>
> STOP. Wait for the user's choice. Do not proceed until they answer.

**No config found -> skip this whole step silently.**
**Config found but user has not answered yet -> STOP. Do not run anything.**

**Run according to the user's choice**, then report results. **Do not report
"all passed" based on exit code alone** - check whether checks with
prerequisites were actually run or silently skipped:

- **All passed and nothing was skipped:**
  > "所有校验通过。
  > 下一步建议：运行 /prune-review 做 change 级代码审查。"
- **Exit code 0 but some checks were skipped (prerequisites unmet):**
  > "⚠️ 校验退出码为 0，但以下校验被跳过（前置条件未满足）：
  > - {校验项}: 需设 {ENV_VAR}，当前未设，已被跳过
  >
  > 虽然退出码为 0，但这些校验未实际执行。请设置前置条件后重跑，或明确接受"未覆盖"。
  > 下一步建议：运行 /prune-review 做 change 级代码审查。"
  Do NOT collapse this into "all passed" - a skipped integration test is not a
  passing integration test. This is the most dangerous false-green: the user
  thinks integration is verified when it was never run.
- **Some failed:**
  > "以下校验未通过：
  > - {校验项}: {失败摘要}
  > - ...
  >
  > 建议先修复失败项再进入 review。是否现在修复，还是先跳过校验进 review？"
  If the user chooses to fix, fix and re-run only the failed checks. If the user
  chooses to skip, proceed to the suggestion below.
- **User chose skip:**
  > "已跳过校验。
  > 下一步建议：运行 /prune-review 做 change 级代码审查。"

Then, based on `per_task_review` in the state file:

- **`final` (the default)** - prompt: "Run `/prune-review` now for the
  **change-level review** (range = base..HEAD) - the final independent backstop
  you chose at startup. Findings are fixed per `fix_mode`."
- **`every`** - each task already had a subagent review; prompt: "All tasks
  reviewed. You can optionally run `/prune-review` for a whole-change review, or
  `/harvest-archive` to archive."
- **`none`** - only self-review was done; prompt: "Run `/prune-review` for a
  **change-level review** (range = base..HEAD) or a **project-level review**
  (whole-codebase health check)."

If there is no git, note that change-level review degrades to the whole working
tree (imprecise); project-level review is unaffected.

## Guardrails

- **Delete code written before tests.** Iron Law. No exceptions (in TDD mode).
- **One test at a time.** Don't write multiple tests before implementing (in TDD mode).
- **Verify RED every time.** Never skip verification (in TDD mode).
- **Mark tasks one at a time.** Replace the exact line, never batch regex across the whole file.
- **Dirty working tree must be asked about.** Never proceed without an explicit user choice.
- **Resume check before overwriting state.** If a state file exists, cross-check it against tasks.md and git log; if inconsistent or incomplete, ask the user how to resume - never blindly overwrite or blindly continue.
- **No-git must be explained and asked about.** Never silently continue.
- **Review is always an independent read-only subagent** (when enabled). grow-apply only triggers and fixes; it never acts as the subagent reviewer.
- **Self-review is always the main conversation, always inline.** Every task gets it before checkpoint commit; its fixes are always inline (fix_mode does not apply).
- **Only append commits, never rewrite history.** No amend, no default squash.
- **A checkpoint commit must precede subagent reviewable work** (skipped under no-git degradation).
- **Ask the three questions one at a time.** Each option explains what happens if chosen - no bare jargon. Defaults: test-driven, subagent review = final sweep, fix in conversation.
- **Fix mode is asked once** (at start) when subagent review is enabled (per-task or final); reused for every subagent review - never re-asked. Skipped when `per_task_review: none`.
- **After fixing, control returns to the user** to decide on re-review - never auto-loop review.
- **Advance `last_completed_task` only when marking `[x]`**, not at checkpoint commit - so a committed-but-unmarked task is detectable as "review unfinished" on resume.
- **Pause on ambiguity.** Don't guess - ask.
- **Communicate in the user's language.** Match the language the user writes in throughout the session. Never mix languages in a single response.
