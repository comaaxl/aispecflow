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

### Step 4: Verify review

Confirm that code review has been completed and all Critical + Important issues are resolved. If not:
> "This change hasn't been reviewed yet. Run `/prune-review` first? Archiving without review means issues may go unnoticed."

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

### Step 6: Requirements document

Check if `docs/requirements.md` exists:
> "`docs/requirements.md` exists. Archive options:
> 1. **Archive it** — copy to `openspec/changes/archive/<change>/requirements.md` and delete the original (clean start for next change)
> 2. **Archive, keep original** — copy but leave the original in `docs/`
> 3. **Skip** — leave as-is"

Do NOT assume whether this file was used for the current change — let the user decide.

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
**Requirements:** Archived / Archived (kept original) / Skipped

Prompt the user: "Change archived. Run `/renew-docs` to update project documentation."
```

## Guardrails

- **Always let the user select the change.** Never auto-select.
- **Always prompt for spec sync.** Don't silently skip it.
- **Always offer spec sync options.** Show all three: Sync now / Skip / Cancel.
- **Never assume requirements.md is related to this change.** Ask the user.
- **Don't delete anything without confirmation.** Requirements archive requires explicit user choice.
- **Communicate in the user's language.** Match the language the user writes in throughout the session. Never mix languages in a single response.
