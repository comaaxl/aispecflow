# Apply State File

grow-apply records its progress in a state file so review ranges can be derived
and progress can survive compaction.

## Location

```
openspec/changes/<change>/.aispecflow-apply-state.md
```

Lives inside the change directory. OpenSpec's `archive` command moves the whole
change directory to `archive/`, so this file travels with it - the CLI does not
strip unknown files (verified against OpenSpec 1.4.1). Because `openspec/` is
checked into git by default, the file is version-controlled and survives
`git clean` and context compaction.

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

Conversation memory does not survive compaction. After a compaction or resume,
recover progress by cross-checking three sources:

1. This state file - `base`, `head`, chosen modes.
2. `tasks.md` checkboxes - which tasks are marked `[x]`.
3. `git log` - the actual commits that exist.

Trust the state file and `git log` over recollection. Tasks listed as complete
in `tasks.md` AND backed by a checkpoint commit in `git log` are done; resume at
the first task not meeting both conditions.

## No-Git Handling

This file is not a substitute for git. Do not simulate commits or invent
boundaries when git is absent. With `base: no-git`:

- Task-level review is unavailable (no commit boundaries).
- Change-level review degrades to the whole working tree (imprecise).
- Project-level review is unaffected (it reads the whole codebase anyway).
- Recovery relies only on `tasks.md` checkboxes.

Recommend `git init` to the user when no git is detected.
