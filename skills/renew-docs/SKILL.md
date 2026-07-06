---
name: renew-docs
description: Update project documentation to stay in sync with the current codebase. Always backs up before modifying. Use anytime docs are stale, not just after archive.
---

# Refresh Docs — Keep Documentation Current

Update project-level documentation to reflect the current state of the codebase. **Never modify without user consent. Always backup before editing.**

## When to Use

- After running `/harvest-archive`
- When you notice README.md or `docs/project-overview.md` are outdated
- After major changes or long periods without doc updates
- User says "update the docs" or "refresh documentation"

## Process

### Step 1: Choose refresh mode

> "How should I refresh documentation?
> 1. **Based on a specific change** — update only sections affected by a change (active or archived)
> 2. **Full refresh** — scan entire codebase and sync all docs to current reality"

If the user picks **mode 1**, list available changes:
- Active: from `openspec list --json`
- Archived: from `openspec/changes/archive/`
- Let the user choose one

### Step 2: Understand what changed

**Mode 1 (change-based):**
Read the selected change's artifacts:
```
openspec/changes/archive/YYYY-MM-DD-<name>/proposal.md
openspec/changes/archive/YYYY-MM-DD-<name>/design.md
```
(or from active change if not yet archived)

Extract: new capabilities, modified behavior, removed features, new dependencies, architecture changes.

**Mode 2 (full refresh):**
Re-scan the codebase similar to `/terrain-scan`:
- Current directory structure and modules
- Entry points and API endpoints
- Dependencies and tech stack
- Configuration and environment variables
- Key files

### Step 3: Diff & identify gaps

Compare what exists against each target document:

**`README.md`:**
- Features list current?
- Setup/install steps still correct?
- Usage and API examples up to date?
- Environment variables documented?

**`docs/project-overview.md`:**
- Architecture section reflects current modules?
- Directory map includes all directories?
- Tech stack accurate?
- Key files index complete?

### Step 4: Propose changes — get consent

**Before touching any file**, present proposed changes:

> "I've identified the following documentation gaps:
>
> **Mode 1** (from change `<name>`):
> **README.md**:
> - Add: Feature X under 'Features'
> - Update: Setup — new env var `FOO_BAR`
> **docs/project-overview.md**:
> - Update: Directory map — new `src/foo/`
> - Update: Tech stack — added `some-library`
>
> **Mode 2** (full refresh):
> **README.md**:
> - 3 new features not listed
> - Setup instructions outdated (step 4 changed)
> - 2 new env vars missing
> **docs/project-overview.md**:
> - Architecture section misses 2 new modules
> - Tech stack has 4 outdated entries
>
> May I apply these changes? (Documents will be backed up before editing.)"

**Never modify without explicit user consent.**

### Step 5: Backup & apply

For each document the user approved:

```bash
mkdir -p docs/.archive && cp <document> docs/.archive/<document-name>.$(date +%Y%m%d-%H%M%S).bak
```

Apply the approved changes. Keep edits minimal — only what's needed to sync with reality. Match existing formatting and tone.

### Step 6: Show final state

```
## Docs Refreshed

**Mode:** Change-based (from `<name>`) / Full refresh
**Documents updated:**
- README.md — added feature X, updated setup instructions
- docs/project-overview.md — updated directory map, tech stack

**Backups:**
- docs/.archive/README.md.20260705-143000.bak
- docs/.archive/project-overview.md.20260705-143001.bak

All documentation now reflects the current codebase.
```

## Guardrails

- **Ask mode before scanning.** Change-based or full refresh.
- **Never modify without user consent.** Propose changes first, wait for approval.
- **Always backup first.** `mkdir -p docs/.archive && cp file docs/.archive/<name>.$(date +%Y%m%d-%H%M%S).bak` before any edit.
- **Minimal edits only.** Don't rewrite sections that didn't change.
- **Don't invent.** If you can't verify a change from the code, don't add it.
- **Respect document style.** Match existing formatting and tone.
- **Communicate in the user's language.** Match the language the user writes in throughout the session. Never mix languages in a single response.
