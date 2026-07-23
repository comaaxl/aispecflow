---
name: seed-grill
description: Relentlessly interview to clarify what the user wants, then produce the documents they need. Output is intent-driven - requirements, PRD, architecture, prototype, API, or whatever the user asks for.
---

# Grill - Intent-Driven Interview & Document Generation

Relentlessly interview the user until what they want is unambiguous and complete, then produce the document(s) they need. This is the hard gate before any spec or code gets written.

**Core principle**: The output is driven by the user's intent, not by a fixed default. Listen first - understand what the user wants this round (requirements? architecture? prototype? API design?), then interview toward that goal, then produce it.

**CRITICAL - override default behavior**: This skill is an interactive interview. You MUST ask questions one at a time in the conversation and wait for the user's reply before proceeding. Do NOT batch questions. Do NOT make assumptions and skip ahead. Do NOT generate any document until the user has answered enough questions to reach saturation. If you feel tempted to "just proceed with reasonable assumptions" - STOP. That is the wrong mode for this skill. The entire value of grilling comes from the back-and-forth dialogue, not from the document at the end.

## Step 0: Understand Intent

Before any interview, understand what the user wants this round. Read the user's message and determine:

- What document(s) do they want? (requirements, PRD, technical architecture, API design, UI prototype, or something else)
- Are they communicating a need for requirements clarification? Or do they want a specific artifact like a prototype or architecture?
- Could they want multiple documents? (e.g., "I want to start a new project" might need requirements + architecture + prototype)

If the intent is clear from the user's message, proceed. If not, ask one question to clarify:

> "What would you like me to produce? For example: requirements document, PRD, technical architecture, API design, UI prototype, or something else?"

The user's intent drives everything that follows - which questions to ask, what to produce at the end. There is no default output. If the user is discussing requirements, produce requirements. If they want a prototype, produce a prototype. If they want both, produce both.

**For any document produced**: follow professional engineering conventions. Consult `references/document-types.md` for common document types with their key questions and structure suggestions - this is a reference, not a rigid template. Adapt to the project's actual needs.

## Pre-flight

Before asking the first question:

0. **Confirm the output path.** Check if cwd contains a subdirectory with project code (pyproject.toml, package.json, go.mod, etc.). If yes:
     > "Project code is in `<subdir>/`. Where should I place generated docs?
     > 1. `<cwd>/<subdir>/docs/` (alongside the code)
     > 2. `<cwd>/docs/` (project root)"
     Default to option 1. Use the confirmed path for all file writes.

1. **Explore the codebase directly.** Read relevant source files, configs, and entry points. The code is the ultimate source of truth. If docs exist (README, CONTEXT.md, project-overview.md), read them as context - but never trust them over what the code actually says. If code and docs conflict, the code wins.

2. **Read `CONTEXT.md`** if it exists - respect existing domain terminology. If the user uses a term that conflicts with the glossary, surface it immediately.

3. **Read relevant ADRs** in `docs/adr/` - don't re-litigate settled decisions.

4. **Read `docs/project-overview.md`** if it exists - but NEVER assume it matches the current codebase. Spot-check key claims (directory structure, entry points, tech stack versions, deps) against the actual files. If discrepancies found, note them and prefer the code. Treat it as potentially stale, not gospel.

## The Interview

```
Interview me relentlessly about every aspect of this plan until we reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.

Ask the questions one at a time, waiting for feedback on each question before continuing. Asking multiple questions at once is bewildering.

If a question can be answered by exploring the codebase, explore the codebase instead.
```

### Question Domains

Walk through these domains in order, but don't force every question if the answer is clear. Skip domains that don't apply. Adapt the depth and focus based on the user's intent - if they want a UI prototype, spend more time on user flows and interactions; if they want a technical architecture, spend more time on component boundaries and tech trade-offs. Use `references/document-types.md` for type-specific question suggestions.

**1. Problem & Motivation**
- What problem does this solve? Who has this problem?
- Why now? What's the trigger?
- What happens if we don't build this?

**2. Users & Stakeholders**
- Who will use this? What do they currently do?
- Are there non-human consumers (other systems, APIs)?

**3. Success Criteria**
- How do we know this is done? What does "working" look like?
- What metrics would tell us this is successful?

**4. Scope & Boundaries**
- What's explicitly in? What's explicitly out?
- What's the MVP vs. future iterations?
- What assumptions are we making?

**5. Constraints**
- Technical constraints (language, framework, database, deployment)?
- Business constraints (deadline, budget, compliance)?
- Existing system constraints (must integrate with X, can't break Y)?

**6. Risks & Edge Cases**
- What could go wrong? What's the worst case?
- What edge cases keep you up at night?
- What happens under load? Under failure?

**7. Dependencies & Sequencing**
- What needs to be built first? What blocks what?
- What external dependencies exist? Are they reliable?

### During the Interview

- **Challenge fuzzy language**: "You said 'fast' - what response time? Under what load?"
- **Surface contradictions**: "Earlier you said X, but now you're describing Y - which is correct?"
- **Sharpen with scenarios**: "Imagine a user does X and then Y happens - what should the system do?"
- **Recommend and move on**: Provide a suggested answer for each question, get confirmation, move to the next.

## Documentation - Write As You Go

**Do not batch these up at the end.** Write them inline as decisions crystallize.

### CONTEXT.md (Domain Glossary)

Each time a term is resolved, update `CONTEXT.md` using this format:

```markdown
# <Project Name>

## Language

**Term**: One or two sentence definition.
_Avoid_: synonym1, synonym2
```

Rules:
- Create `CONTEXT.md` at the project root if it doesn't exist.
- Be opinionated - pick the best term, list alternatives under `_Avoid_`.
- Only include project-specific domain terms, not general programming concepts.

### docs/adr/ (Architectural Decision Records)

Offer an ADR when ALL three are true:
1. **Hard to reverse** - changing later is costly
2. **Surprising without context** - a future reader would wonder "why?"
3. **Real trade-off** - genuine alternatives existed

Format (`docs/adr/NNNN-title.md`):

```markdown
# <Short title>

<1-3 sentences: context, decision, why.>
```

Number sequentially. Only include optional sections (Status, Considered Options, Consequences) when they add genuine value.

## Final Output: Produce What the User Asked For

Once the interview reaches saturation - the user says "that's everything" and you can't find more ambiguity - produce the document(s) the user asked for. All documents go under `docs/`.

### Check for existing documents

Before writing, check if any target document already exists in `docs/`. If it does:

> "`docs/<filename>` already exists (last modified: <date>, likely from a previous session).
>
> 1. **Overwrite it** - replace with this round's content
> 2. **Backup, then overwrite** (recommended) - copy existing to `docs/.archive/<filename>.<date>.bak` first, then write new
> 3. **Save to a different path** - e.g., `docs/<filename>-<topic>.md`"

Let the user choose. Default to option 2 if they don't specify. Create `docs/.archive/` if it doesn't exist when using option 2.

### Write the document(s)

Produce the document(s) using the interview context. Follow professional engineering conventions - `references/document-types.md` provides common document types with key questions and structure suggestions, but the actual structure should adapt to the project's needs.

Present the document(s) to the user. Say: "This is my understanding. Does this capture everything correctly?"

If they request changes, iterate.

### Common Document Types

When the user asks for a specific type of document, consult `references/document-types.md` for:
- Key questions that should have been covered in the interview
- Structure suggestions for the output document

This is a reference guide, not a rigid template. The user may ask for document types not listed there - use your judgment and follow professional conventions.

### Handoff

After the user confirms the document(s):

> "Document(s) confirmed. Run `/bloom-spec` when you're ready to generate the formal specs and start implementation. Or let me know if you need anything else."

Then **STOP completely.** Do NOT start coding, designing, or implementing anything.

### Commit artifacts (if git)

After the user confirms the document(s), if the project has git, commit the
artifacts this skill produced so the working tree is clean for the next phase.
**Only commit files this skill created or updated** - do not `git add -A` or
sweep up unrelated dirty files.

Files this skill may have produced (depends on user intent):
- `docs/` - whatever document(s) the user asked for (requirements.md, prd.md,
  tech-architecture.md, api-design.md, prototype files, etc.)
- `CONTEXT.md`
- `docs/adr/NNNN-*.md` (any ADRs created during grilling)

List the exact files to the user and confirm before committing:

> "I'll commit these grill-phase artifacts:
> - docs/requirements.md
> - docs/tech-architecture.md
> - CONTEXT.md
> - docs/adr/0001-db-selection.md
>
> Commit as `docs(seed-grill): <topic>`. OK?"

If the user confirms, commit (append-only, never amend). If no git, or the user
declines, skip silently. Do not block the handoff on this.

## Guardrails

- **One question at a time. Always.**
- **Intent drives output.** There is no default document. Understand what the user wants first, then interview toward it, then produce it.
- **Don't skip the documentation.** Inline updates to CONTEXT.md and ADRs are non-negotiable.
- **Follow professional engineering conventions.** Consult `references/document-types.md` for guidance, but adapt to the project's actual needs.
- **Explore codebase when relevant.** If `docs/project-overview.md` exists, use it as context but verify against the code.
- **STOP after user confirms the document(s).** Do NOT start coding, implementing, or touching any source files.
