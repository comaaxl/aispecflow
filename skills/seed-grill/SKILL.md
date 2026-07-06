---
name: seed-grill
description: Relentlessly interview to sharpen requirements before building. Produces a requirements document, domain glossary, and architectural decision records.
---

# Grill — Requirements Elicitation

Relentlessly interview the user about what they want to build until their requirements are unambiguous and complete. This is the hard gate before any spec or code gets written.

**Core principle**: One question at a time. Walk every branch of the decision tree. Don't let fuzzy language survive.

**CRITICAL — override default behavior**: This skill is an interactive interview. You MUST ask questions one at a time in the conversation and wait for the user's reply before proceeding. Do NOT batch questions. Do NOT make assumptions and skip ahead. Do NOT generate the requirements document until the user has answered enough questions to reach saturation. If you feel tempted to "just proceed with reasonable assumptions" — STOP. That is the wrong mode for this skill. The entire value of grilling comes from the back-and-forth dialogue, not from the document at the end.

## Pre-flight

Before asking the first question:

0. **Confirm the output path.** Check if cwd contains a subdirectory with project code (pyproject.toml, package.json, go.mod, etc.). If yes:
     > "Project code is in `<subdir>/`. Where should I place generated docs?
     > 1. `<cwd>/<subdir>/docs/` (alongside the code)
     > 2. `<cwd>/docs/` (project root)"
     Default to option 1. Use the confirmed path for all file writes.

1. **Explore the codebase directly.** Read relevant source files, configs, and entry points. The code is the ultimate source of truth. If docs exist (README, CONTEXT.md, project-overview.md), read them as context — but never trust them over what the code actually says. If code and docs conflict, the code wins.

2. **Read `CONTEXT.md`** if it exists — respect existing domain terminology. If the user uses a term that conflicts with the glossary, surface it immediately.

3. **Read relevant ADRs** in `docs/adr/` — don't re-litigate settled decisions.

4. **Read `docs/project-overview.md`** if it exists — but NEVER assume it matches the current codebase. Spot-check key claims (directory structure, entry points, tech stack versions, deps) against the actual files. If discrepancies found, note them and prefer the code. Treat it as potentially stale, not gospel.

## The Interview

```
Interview me relentlessly about every aspect of this plan until we reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.

Ask the questions one at a time, waiting for feedback on each question before continuing. Asking multiple questions at once is bewildering.

If a question can be answered by exploring the codebase, explore the codebase instead.
```

### Question Domains

Walk through these domains in order, but don't force every question if the answer is clear. Skip domains that don't apply.

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

- **Challenge fuzzy language**: "You said 'fast' — what response time? Under what load?"
- **Surface contradictions**: "Earlier you said X, but now you're describing Y — which is correct?"
- **Sharpen with scenarios**: "Imagine a user does X and then Y happens — what should the system do?"
- **Recommend and move on**: Provide a suggested answer for each question, get confirmation, move to the next.

## Documentation — Write As You Go

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
- Be opinionated — pick the best term, list alternatives under `_Avoid_`.
- Only include project-specific domain terms, not general programming concepts.

### docs/adr/ (Architectural Decision Records)

Offer an ADR when ALL three are true:
1. **Hard to reverse** — changing later is costly
2. **Surprising without context** — a future reader would wonder "why?"
3. **Real trade-off** — genuine alternatives existed

Format (`docs/adr/NNNN-title.md`):

```markdown
# <Short title>

<1-3 sentences: context, decision, why.>
```

Number sequentially. Only include optional sections (Status, Considered Options, Consequences) when they add genuine value.

## Final Output: requirements.md

### Check for existing requirements

Before writing, check if `docs/requirements.md` already exists. If it does:

> "`docs/requirements.md` already exists (last modified: <date>, likely from a previous change).
>
> 1. **Overwrite it** — replace with this change's requirements
> 2. **Backup, then overwrite** (recommended) — copy existing to `docs/.archive/requirements.md.<date>.bak` first, then write new
> 3. **Save to a different path** — e.g., `docs/requirements-<change-name>.md`"

Let the user choose. Default to option 2 if they don't specify. Create `docs/.archive/` if it doesn't exist when using option 2.

### Write the document

Once the interview reaches saturation — the user says "that's everything" and you can't find more ambiguity — produce `docs/requirements.md`.

```markdown
# Requirements: <Feature/Change Name>

## Problem Statement
<1-2 sentences>

## Users & Stakeholders
<Who and their needs>

## Functional Requirements
- <Requirement 1>: <description>
- <Requirement 2>: <description>
...

## Non-Functional Requirements
- Performance: <specific targets>
- Security: <requirements>
- ...

## Success Criteria
- <Measurable outcome>
...

## Scope Boundaries
**In scope**: ...
**Out of scope**: ...
**Assumptions**: ...

## Risks & Mitigations
- <Risk> → <Mitigation>

## Dependencies
- <What this depends on and sequencing>
```

Present this document to the user. Say: "This is my understanding of what we're building. Does this capture everything correctly?"

Once the user confirms requirements.md, **STOP completely.** Do NOT start coding, designing, or implementing anything. Your ONLY next action is to prompt:

> "Requirements confirmed. Run `/bloom-spec` when you're ready to generate the formal specs." 

## Guardrails

- **One question at a time. Always.**
- **Don't skip the documentation.** Inline updates to CONTEXT.md and ADRs are non-negotiable.
- **Explore codebase when relevant.** If `docs/project-overview.md` exists, use it as context but verify against the code.
- **STOP after user confirms requirements.md.** Do NOT start coding, implementing, or touching any source files. Your only output is a prompt to run `/bloom-spec`.
