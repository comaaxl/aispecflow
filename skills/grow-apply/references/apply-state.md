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
per_task_review: on|off  # whether task-level review fires after each task
fix_mode: inline|subagent  # how review findings get fixed
head: <sha>            # most recent checkpoint commit (for task-level range)
---
```

- `base` - the starting point of this change. `BASE..HEAD` is the change-level
  review range. `no-git` means git is unavailable; change-level review degrades
  to the whole working tree.
- `head` - the most recent checkpoint commit. Updated after each task's
  checkpoint commit. The current task's review range is captured at commit time
  as `TASK_BASE..HEAD` (where `TASK_BASE` is the commit before this task's
  changes - i.e. the previous `head`). After commit, `head` advances to the new
  commit so the next task's `TASK_BASE` is this commit.
- `per_task_review` - `on` (default when git is available) means every task
  triggers a task-level review after its checkpoint commit. Forced `off` when
  there is no git (no task commit boundaries to review against).

## Read/Write Timing

- **grow-apply start**: write the file with `base`, `started`, `mode`,
  `per_task_review`, `fix_mode`. `head` is left unset until the first checkpoint.
- **After each checkpoint commit**: update `head` to the new commit's SHA.
- **Archive**: the file moves into `archive/` with the change. No manual cleanup.

## Recovery Semantics

Conversation memory does not survive compaction or an interrupted session. When
`/grow-apply` restarts and finds an existing state file, it runs a **resume
check** (see the SKILL.md Step 0): cross-check three sources -

1. This state file - `base`, `head`, chosen modes.
2. `tasks.md` checkboxes - which tasks are marked `[x]`.
3. `git log <base>..HEAD` - the actual `task(...)` / `fix(review-task-...)`
   commits that exist.

**What is recoverable:** which tasks are complete (a task marked `[x]` AND backed
by a checkpoint commit is done). The boundary between completed and incomplete
tasks is reliable.

**What is NOT recoverable:** the in-progress state of the interrupted task
itself. If a task was interrupted mid-implementation (half-written code) or
mid-fix (some review findings fixed, some not), that intra-task state is not
recorded anywhere. The state file and git log only show task-level boundaries.
Restarting re-implements the interrupted task from scratch - first checking the
working tree for half-finished changes to commit or discard, not silently
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
