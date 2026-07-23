---
name: harvest-archive
description: Archive a completed OpenSpec change after implementation and review are done. Syncs delta specs to main specs and moves the change to the archive.
---

# Archive — Complete & Archive Change

Archive a completed change. Syncs spec changes back to main specs and moves the change to the archive directory.

**Prerequisite**: All tasks completed, code review approved.

## Process

### Step 1: Select the change

Run `openspec list --json` to get available changes. **Always let the user choose** — do NOT auto-select, even if only one change exists. Show active changes with their schema.

Announce: "Archiving change: `<name>`"

### Step 2: Verify completion

```bash
openspec status --change "<name>" --json
```

Check:
- All artifacts are `done`
- Parse `artifactPaths.specs.existingOutputPaths` to find delta specs

**If any artifacts are not `done`**: warn the user and ask for confirmation before proceeding.

### Step 3: Check task completion

Read the tasks file (typically `tasks.md`). Count `- [ ]` (incomplete) vs `- [x]` (complete).

**If incomplete tasks found**: warn the user and ask for confirmation.

**If no tasks file**: proceed without task-related warning.

### Step 4: Review reminder (soft, non-blocking)

Check whether code review has been done. This is a reminder, not a gate - you can
still archive without review. If review hasn't been done:
> "This change hasn't been reviewed yet. It's recommended to run
> `/prune-review` (change-level or project-level) first. You can still archive
> without review, but issues may go unnoticed and be carried into the archive.
> Proceed with archiving, or review first?"

Let the user decide. Do not block archiving if they choose to proceed.

### Step 5: Assess delta spec sync state

Use `artifactPaths.specs.existingOutputPaths` from status JSON to find delta specs. If none exist, skip to Step 6.

**If delta specs exist:**
- Compare each delta spec with its corresponding main spec at `openspec/specs/<capability>/spec.md`
- If the main spec already exists, treat this as a modification to an existing capability
- If the main spec does **not** exist, treat this as a **new capability** that can be created in `openspec/specs/<capability>/spec.md` during sync
- Determine what changes would be applied (additions, modifications, removals, renames)
- Show a combined summary before prompting

**Prompt options:**
- "Sync now (recommended)" — apply delta specs to main specs
- "Archive without syncing" — move to archive but don't update main specs
- "Cancel" — abort the archive

If user chooses sync, delta specs are synced to main specs as part of the archive process.

### Step 6: Archive grill-phase documents

Scan `docs/` for all documents. Present the list to the user and ask which should
be archived with this change:

> "I found these documents in `docs/`:
> - `docs/requirements.md` (last modified: <date>)
> - `docs/prd.md` (last modified: <date>)
> - `docs/tech-architecture.md` (last modified: <date>)
> - ... (all files found, excluding `docs/adr/` and `docs/.archive/`)
>
> Which should be archived with this change? For each:
> 1. **Archive** - copy to `openspec/changes/archive/<change>/` and delete the original (clean start for next change)
> 2. **Archive, keep original** - copy but leave the original in `docs/`
> 3. **Skip** - leave as-is"

Let the user decide per document. Some documents (e.g., `tech-architecture.md`,
`project-overview.md`) are project-level and typically should NOT be archived -
they persist across changes. Others (e.g., `requirements.md`) are per-change and
typically should be archived. Do NOT assume - let the user decide for each.

Do not include `docs/adr/` or `docs/.archive/` in the scan - ADRs are permanent
records and backups are not documents.

### Step 7: Perform the archive

```bash
openspec archive "<name>"
```

The CLI moves the change to `openspec/changes/archive/YYYY-MM-DD-<name>/`.

### Step 8: Confirm

```
## Archive Complete

**Change:** <name>
**Archived to:** openspec/changes/archive/YYYY-MM-DD-<name>/
**Specs:** ✓ Synced to main specs (or "No delta specs" or "Sync skipped")
**Documents:** <list of archived documents>

Prompt the user: "Change archived. Run `/renew-docs` to update project documentation."
```

### Step 9: Commit archive changes (if git)

After archiving, if the project has git, commit the files this step moved or
synced so the working tree is clean. **Only commit files this skill touched** -
do not `git add -A` or sweep up unrelated dirty files.

Files this skill may have changed:
- `openspec/changes/archive/YYYY-MM-DD-<name>/` (the moved change directory)
- `openspec/changes/<name>/` (removed after move)
- `openspec/specs/` (synced delta specs)
- `docs/` documents that were archived (moved or copied)

List the exact paths to the user and confirm before committing:

> "I'll commit these archive changes:
> - openspec/changes/archive/2026-07-22-<name>/ (moved change)
> - openspec/specs/ (synced delta specs)
>
> Commit as `chore(harvest-archive): archive <name>`. OK?"

If the user confirms, commit (append-only, never amend). If no git, or the user
declines, skip silently. Do not block the handoff to `/renew-docs` on this.

## Guardrails

- **Always let the user select the change.** Never auto-select.
- **Always prompt for spec sync.** Don't silently skip it.
- **Always offer spec sync options.** Show all three: Sync now / Skip / Cancel.
- **Never assume which documents are related to this change.** Ask the user for each document.
- **Don't delete anything without confirmation.** Document archival requires explicit user choice per file.
- **Communicate in the user's language.** Match the language the user writes in throughout the session. Never mix languages in a single response.
