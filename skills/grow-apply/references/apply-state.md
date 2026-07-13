# Apply State File

grow-apply records its progress in a state file so review ranges can be derived
and progress can survive compaction.

## Location

```
openspec/changes/<change>/.aispecflow-apply-state.md
```

Lives inside the change directory and travels with it into `archive/` when the
change is archived.

## Fields

```yaml
---
base: <sha>            # HEAD at grow-apply start, or "no-git" if no git
started: <date>        # when grow-apply started
mode: tdd|straight     # implementation mode chosen by the user
per_task_review: every|final|none  # subagent review depth (self-review is always on)
fix_mode: inline|subagent  # how subagent review findings get fixed (unset when none)
head: <sha>            # most recent checkpoint commit
last_completed_task: 1.3  # last task marked [x] (empty until first task completes)
---
```

- `base` - the starting point of this change. `BASE..HEAD` is the change-level
  review range. `no-git` means git is unavailable; change-level review degrades
  to the whole working tree.
- `head` - the most recent checkpoint commit. Updated after each task's
  checkpoint commit. The current task's review range is captured at commit time
  as `TASK_BASE..HEAD` (where `TASK_BASE` is `git rev-parse HEAD` before the
  commit - i.e. the previous `head`). `head` is primarily for resume recovery
  and the change-level review range; `TASK_BASE` is always read live from git,
  not from this field (more robust - doesn't depend on state-file consistency).
- `per_task_review` - subagent review depth:
  - `every` - subagent reviews after each task (self-review + subagent per task)
  - `final` (default when git is available) - only self-review per task; one
    subagent reviews the whole change at the end
  - `none` - only self-review; no subagent
  - Forced `none`/`final` when there is no git (`every` needs commit boundaries).
- `last_completed_task` - the number of the last task marked `[x]` (e.g. `1.3`).
  Advanced **only when the task is marked `[x]`**, NOT at checkpoint commit - so
  a committed-but-unmarked task (review unfinished) is detectable on resume.
  Empty until the first task completes.

## Read/Write Timing

- **grow-apply start**: write the file with `base`, `started`, `mode`,
  `per_task_review`, `fix_mode` (unset if `none`). `head` and
  `last_completed_task` are left empty until the first checkpoint / completion.
- **After each checkpoint commit**: update `head` to the new commit's SHA.
- **When a task is marked `[x]`**: advance `last_completed_task` to that task's
  number.
- **Archive**: the file moves into `archive/` with the change. No manual cleanup.

## Recovery Semantics

Conversation memory does not survive compaction or an interrupted session. When
`/grow-apply` restarts and finds an existing state file, it runs a **resume
check** (see the SKILL.md Step 0): cross-check four sources -

1. This state file - `base`, `head`, `last_completed_task`, chosen modes.
2. `tasks.md` checkboxes - which tasks are marked `[x]`.
3. `git log <base>..HEAD` - the actual `task(...)` / `fix(review-task-...)`
   commits that exist.
4. **Cross-validate `last_completed_task`** against tasks.md: if the state file
   says task X is complete but tasks.md does not mark X as `[x]` (or vice
   versa), the state is inconsistent - stop and ask the user.

**What is recoverable:** which tasks are complete (a task marked `[x]` AND
backed by a checkpoint commit AND reflected in `last_completed_task` is done).
The boundary between completed and incomplete tasks is reliable.

**Detectable "half-done" state:** if `git log` has a `task(X.Y)` commit but
tasks.md does not mark X.Y as `[x]` and `last_completed_task` is still the
previous task, then X.Y was committed but its review was not finished. Stop and
ask: continue the review, or discard the commit and redo.

**What is NOT recoverable:** the in-progress state of the interrupted task
itself. If a task was interrupted mid-implementation (half-written code) or
mid-self-review (some inline fixes done, some not), that intra-task state is not
recorded anywhere. Restarting re-implements the interrupted task from scratch -
first checking the working tree for half-finished changes to commit or discard,
not silently
carrying them forward.

Trust the state file and `git log` over recollection. If the three sources are
inconsistent or there are incomplete tasks, ask the user how to resume - do not
blindly overwrite the state file or blindly continue.

## No-Git Handling

This file is not a substitute for git. Do not simulate commits or invent
boundaries when git is absent. With `base: no-git`:

- Task-level review is unavailable (no commit boundaries).
- Change-level review degrades to the whole working tree (imprecise).
- Project-level review is unaffected (it reads the whole codebase anyway).
- Recovery relies only on `tasks.md` checkboxes.

Recommend `git init` to the user when no git is detected.
