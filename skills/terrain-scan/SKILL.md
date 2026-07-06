---
name: terrain-scan
description: Systematically scan a codebase to produce a comprehensive project overview document. Use when the user wants to understand a project's architecture, module structure, conventions, and capabilities before starting development work.
---

# Repository Discovery

Systematically explore the codebase and produce a single, comprehensive **`docs/project-overview.md`** that gives any AI agent (and developer) a complete mental model of the project in one read.

**Core principle**: Ground every statement in real code. Never guess. Read files, trace imports, follow call chains. If you can't verify a claim, don't include it.

## When to Use

- Starting work on an unfamiliar project
- Before running `seed-grill` or `bloom-spec` for the first time
- When project documentation is outdated or missing
- User says "explore this project" or "give me a project overview"

## Process

### Phase 0: Confirm Output Path

Check if cwd contains a subdirectory with project code (pyproject.toml, package.json, go.mod, etc.). If yes:
> "Project code is in `<subdir>/`. Where should I place `project-overview.md`?
> 1. `<cwd>/<subdir>/docs/` (alongside the code)
> 2. `<cwd>/docs/` (project root)"

Default to option 1.

## Phase 1: Broad Reconnaissance

Read these files first (adjust for project language/framework):
1. `README.md`, `CONTEXT.md`, `AGENTS.md`, `CLAUDE.md` — any existing docs
2. `package.json` / `pyproject.toml` / `Cargo.toml` / `go.mod` — dependencies and metadata
3. Top-level config files (`.env.example`, `docker-compose.yml`, `Makefile`, etc.)

Build a rough map: what kind of project is this, what does it do, what's the tech stack.

### Phase 2: Deep Exploration (Multi-Agent)

**Dispatch parallel subagents** to explore different dimensions of the codebase simultaneously. Each subagent operates on a read-only basis, tracing real code paths.

**Agent 1 — Architecture & Entry Points:**
- Find all entry points (main functions, CLI commands, HTTP routes, MCP tools)
- Trace the startup/bootstrap sequence
- Map external integrations (databases, APIs, message queues)
- Identify the deployment model

**Agent 2 — Module & Directory Map:**
- Walk the directory tree, noting every top-level and second-level directory
- For each module/directory: what responsibility does it have? What does it import? What imports it?
- Note module boundaries and cross-cutting concerns

**Agent 3 — Domain & Data Model:**
- Find all data models, schemas, type definitions
- Trace relationships between entities
- Identify the ubiquitous language (domain terms)
- Note any existing `CONTEXT.md` glossary

**Agent 4 — Conventions & Patterns:**
- Coding style (formatting, naming conventions, file organization)
- Error handling patterns
- Testing patterns and frameworks used
- Dependency injection / configuration patterns
- Any existing `.editorconfig`, `.prettierrc`, `eslint.config.*`, etc.

Each subagent returns a structured summary. The orchestrator (you) merges these into one document.

### Phase 3: Synthesize

Merge the four agent reports into a single `docs/project-overview.md`. Structure:

```markdown
# Project Overview: <project-name>

## 1. Project Purpose
What this project does, who it serves, core value proposition.
2-4 sentences. Cite README if available.

## 2. Tech Stack
Languages, frameworks, databases, infrastructure. One table.

## 3. Architecture
High-level architecture diagram (Mermaid or ASCII). Entry points, request lifecycle, key design decisions. Include any existing ADRs referenced.

## 4. Directory & Module Map
Each top-level directory with a 1-2 sentence responsibility description. Key files within each.

## 5. Domain Model
Key entities, their relationships, and domain terminology. Reference `CONTEXT.md` if it exists; otherwise include glossary inline.

## 6. Conventions & Patterns
Coding style, naming, error handling, testing patterns, configuration management.

## 7. Key Files Index
Critical files with 1-line descriptions: entry points, config, core business logic, key interfaces.

## 8. Known Issues & Technical Debt
Problems visible from code inspection: TODO comments, deprecated APIs, inconsistent patterns, missing tests.
```

### Phase 4: Review & Save

1. **Self-review**: Scan the document for claims not backed by code. Remove any speculation.
2. **Present to user**: Show a summary and ask: "Does this match your understanding of the project? Any corrections?"
3. **Save** to `docs/project-overview.md` (create `docs/` if needed).
4. **Commit suggestion**: Recommend committing this file so it's available to future agent sessions.

## Output

- One file: `docs/project-overview.md`
- User-facing summary of key findings
- Recommendation: "Run `/seed-grill` next if you have a feature to build, or `/bloom-spec` if you already have clear requirements. When the project evolves, use `/renew-docs` to keep this overview up to date."

## Guardrails

- **Read only**. Never modify code during discovery.
- **Verify against code**. Every claim must trace to a real file, import, or config.
- **Prefer concision**. Each section should be skimmable. Don't write a novel.
- **Flag unknowns**. If something can't be determined from code alone (e.g., business rationale), mark it as `[UNKNOWN]`.
- **Respect existing docs**. If `CONTEXT.md` or ADRs exist, reference them, don't duplicate.
- **Communicate in the user's language.** Match the language the user writes in throughout the session. Never mix languages in a single response.
