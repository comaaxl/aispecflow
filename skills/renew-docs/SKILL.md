---
name: renew-docs
description: Update project documentation to stay in sync with the current codebase. Always backs up before modifying. Use anytime docs are stale, not just after archive.
---

# Refresh Docs — Keep Documentation Current

Update project-level documentation to reflect the current state of the codebase. **Never modify without user consent. Always backup before editing.**

## When to Use

- After running `/harvest-archive`
- When you notice project docs are outdated (README, architecture docs, rule
  files, ADRs, deploy docs, changelog, etc.)
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

### Step 2.5: Probe the project's documentation inventory

Before diffing, discover what documentation this project actually has. Do not
assume a fixed set - probe and list whatever exists.

**Probe signals** (a starting point, NOT exhaustive - extend to match what you
find):

| Probe target | If found, include |
|---|---|
| Root `README.md` / `README-zh.md` / `README.*.md` | README (always include if present) |
| `CLAUDE.md` / `AGENTS.md` (and symlinks - check they stay in sync) | Rule files |
| `docs/` directory, all `.md` under it | Project docs |
| `adr/` / `docs/adr/` / `architecture-decisions/` | Architecture Decision Records |
| `CHANGELOG.md` / `HISTORY.md` / `CHANGES.md` | Changelog |
| `DEPLOY.md` / `docs/deploy*` / `docs/ops*` / `RUNBOOK*` | Deploy/ops docs |
| `docs/api*` / `openapi.*` / `docs/openapi*` | API docs |
| `docs/migration*` / schema docs / `db/migrations/README*` | Schema/migration docs |
| Project-specific docs (any `.md` the project treats as authoritative) | Include |

Rules:
- **Dynamic list, not a fixed set.** A project may have 1 doc or 10. List what
  exists, nothing more.
- **No docs found -> stop and tell the user** "No project documentation detected.
  Nothing to refresh." Do not invent documents to update.
- **Never hardcode a required set.** A project without `docs/project-overview.md`
  is fine; a project with only ADRs is fine.

**Knowledge placement guide** (decide where an update belongs, to avoid putting
rule-file content in README or history in docs):

| Knowledge type | Belongs in |
|---|---|
| Boundaries/commands/workflows an Agent will get wrong if not seen | `CLAUDE.md` / `AGENTS.md` (rule files) |
| How to use/operate/maintain the system | `README.md` / `docs/` |
| Historical process, single incidents, version narrative | `CHANGELOG.md` / git / incident docs |
| Architecture decisions and their rationale | ADR |
| Stable mechanisms or lessons that recurred and must be known by successors | Graduate to docs or rule files (not a second copy in memory) |

When a change touches knowledge that spans categories, update each owning
document for its slice - do not duplicate the same content across documents.

### Step 3: Diff & identify gaps

Compare what exists against each probed document, checking by its **document
type** (not a fixed checklist):

**README-type** (README.md / README-zh.md):
- Features list current?
- Setup/install steps still correct?
- Usage and API examples up to date?
- Environment variables documented?

**Architecture-type** (project-overview / ADR):
- Architecture section reflects current modules?
- Directory map includes all directories?
- Tech stack accurate?
- Key files index complete?
- For ADR: does a recent decision need a new ADR, or does an existing ADR now
  contradict the code?

**Rule-file-type** (CLAUDE.md / AGENTS.md):
- Commands, build/run steps, and workflows still match the code?
- Boundaries and conventions still accurate?
- Dead references to removed files/modules?

**Changelog-type** (CHANGELOG / HISTORY):
- Does this change warrant a changelog entry? Is it there?
- Are past entries accurate (not the focus - only flag gross contradictions)?

**Deploy/ops-type** (DEPLOY / RUNBOOK / ops docs):
- Deploy steps, env vars, ports still match code/config?
- Rollback/runbook steps still valid?

**API-type** (api docs / openapi):
- Endpoints, request/response shapes, auth still match code?

**Schema/migration-type**:
- Schema docs match current migrations?
- Migration order and notes accurate?

For any other document type found in the probe, judge by its purpose: does it
still match the current code? Apply the same "verify against reality" standard.

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
> **adr/0007-cache-strategy.md**:
> - New ADR needed: caching layer added but no decision recorded
>
> (List only the documents actually probed in Step 2.5. The above is a sample
> shape - your actual list depends on what the project has.)
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
- README.md - added feature X, updated setup instructions
- docs/project-overview.md - updated directory map, tech stack
- CLAUDE.md - updated run command
- CHANGELOG.md - added entry for this change

**Backups:**
- docs/.archive/README.md.20260705-143000.bak
- docs/.archive/project-overview.md.20260705-143001.bak
- docs/.archive/CLAUDE.md.20260705-143002.bak
- docs/.archive/CHANGELOG.md.20260705-143003.bak

All probed documentation now reflects the current codebase.
```

### Step 7: Commit doc changes (if git)

After applying the approved doc updates, if the project has git, commit the
documents this skill modified. Only commit files this skill edited. Do not
git add -A or sweep up unrelated dirty files.

List the exact paths to the user and confirm before committing. If the user
confirms, commit append-only never amend. If no git or the user declines,
skip silently.


## Guardrails

- **Ask mode before scanning.** Change-based or full refresh.
- **Never modify without user consent.** Propose changes first, wait for approval.
- **Always backup first.** `mkdir -p docs/.archive && cp file docs/.archive/<name>.$(date +%Y%m%d-%H%M%S).bak` before any edit.
- **Minimal edits only.** Don't rewrite sections that didn't change.
- **Don't invent.** If you can't verify a change from the code, don't add it.
- **Respect document style.** Match existing formatting and tone.
- **Communicate in the user's language.** Match the language the user writes in throughout the session. Never mix languages in a single response.
