---
name: bloom-spec
description: Generate formal OpenSpec artifacts (proposal, specs, design, tasks) from requirements. Use when ready to turn requirements into spec-driven-development documents.
---

# Spec — OpenSpec Proposal & Design

Turn validated requirements into formal OpenSpec artifacts: **proposal.md**, **specs/**, **design.md**, and **tasks.md**.

**Prerequisite**: This skill requires the `openspec` CLI to be installed in the project (`npx openspec --version`).

## Phase 0: Confirm Project Root

Check if cwd contains a subdirectory with project code (pyproject.toml, package.json, go.mod, etc.). If yes:
> "Project code is in `<subdir>/`. Where should I work?
> 1. `<cwd>/<subdir>/` (alongside the code)
> 2. `<cwd>/` (project root)"

Default to option 1. Cd into the selected directory. All CLI commands and file reads/writes happen from here.

## Phase 1: Context Discovery

1. **Explore the codebase directly.** Read relevant source files, configs, and entry points. The code is the ultimate source of truth.

2. **Check for supporting documents.** Look for these files and note their last-modified dates:
   - `docs/requirements.md`
   - `docs/project-overview.md`
   - `CONTEXT.md`

   Present the list to the user:
   > "I found these documents that may help as context for the specs:
   > - `docs/requirements.md` (last modified: <date>)
   > - `docs/project-overview.md` (last modified: <date>)
   > - `CONTEXT.md`
   >
   > Use them as context? Or start fresh?"

   If the user chooses to use them, read them. If not, proceed without.

3. **If no requirements document is available** (doesn't exist, or user declines to use it):
   > "No requirements document to work from. Would you like to:
   > 1. Run `/seed-grill` first to clarify requirements
   > 2. Describe what you want to build right now"

4. **ADRs** in `docs/adr/` are referenced as-needed when writing `design.md`. No need to ask upfront.

## Phase 2: Requirement Validation (Ambiguity Check)

Before writing any OpenSpec artifact, scan the requirements for issues:

1. **Ambiguous terms** — "fast", "scalable", "secure", "user-friendly". Flag them.
2. **Missing edge cases** — failures, empty input, concurrency, large data.
3. **Contradictions with existing codebase** — flag with specific file references.
4. **Undefined integrations** — external systems without specified interfaces.
5. **Term mismatch** — conflicts with `CONTEXT.md` glossary.

**If issues found**: Present them grouped by category, one category at a time. Get clarification before proceeding.

**If no issues**: Say "Requirements look solid — generating OpenSpec artifacts now."

## Phase 3: Generate OpenSpec Artifacts

### Step 1: Get the change name

If the user provided a name, use it (convert to kebab-case). Otherwise, ask:
> "What should we call this change? (kebab-case, e.g., `add-user-auth`)"

Do NOT derive or guess — the user must provide it.

### Step 2: Create the change

```bash
openspec new change "<name>"
```

### Step 3: Generate artifacts in sequence

Use `openspec status --change "<name>" --json` to get artifact dependencies, then loop through artifacts in dependency order using `openspec instructions <artifact-id> --change "<name>" --json`.

For each artifact:
- Follow the CLI-returned `instruction` and `template` exactly
- Read completed dependency artifacts for context
- If supporting documents were loaded in Phase 1, use them as context but fill the CLI template — do not invent sections
- Show progress: "Created <artifact-id>"

**Special case — no capability changes declared:**
If the proposal indicates no New or Modified Capabilities, but the current schema still requires specs before tasks can be generated, STOP and ask the user whether they want to:
1. continue with the current spec-driven workflow and formalize a minimal capability spec,
2. revise the proposal so the capability classification is explicit,
3. stop and use a lighter workflow outside this skill.

Do **not** invent a capability name on your own.

**tasks.md format**: Use the standard format from the CLI template. Each task is:
```
- [ ] X.Y Task description
```
 Each task should scope the minimum behavior needed - no speculative or future-proofing tasks (YAGNI).

### Step 4: Show final status

```bash
openspec status --change "<name>"
```

**Output**: change name, artifacts created. Prompt the user: "Specs are ready. Run `/grow-apply` when you want to implement."

### Step 5: Commit artifacts (if git)

After showing the final status, if the project has git, commit the artifacts
this skill produced so the working tree is clean for `/grow-apply` (which checks
for a clean tree at startup). **Only commit files this skill created** - do not
`git add -A` or sweep up unrelated dirty files.

Files this skill produced (under `openspec/changes/<change>/`):
- `proposal.md`
- `design.md`
- `tasks.md`
- `specs/` (spec files)

List the exact files to the user and confirm before committing:

> "I'll commit these spec-phase artifacts:
> - openspec/changes/<change>/proposal.md
> - openspec/changes/<change>/design.md
> - openspec/changes/<change>/tasks.md
> - openspec/changes/<change>/specs/*.md
>
> Commit as `spec(bloom-spec): <change>`. OK?"

If the user confirms, commit (append-only, never amend). If no git, or the user
declines, skip silently. Do not block the handoff to `/grow-apply` on this.

## Guardrails

- **Ask for change name if not provided.** Never derive it.
- **Ask before reading documents.** Never auto-load `docs/requirements.md`.
- **Follow `openspec instructions` exactly.** The CLI owns the artifact format.
- **Validate before generating.** Don't create specs with known ambiguities.
- **Communicate in the user's language.** Match the language the user writes in throughout the session. Never mix languages in a single response.
