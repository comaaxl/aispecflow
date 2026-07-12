# aispecflow

A complete development lifecycle plugin for AI coding agents — from project discovery to docs refresh, all in one plugin.

## Why

Great tools exist for individual stages of development. [Grill Me](https://github.com/mattpocock/skills) sharpens requirements through relentless interviews. [OpenSpec](https://github.com/Fission-AI/openspec) turns specs into trackable artifacts. [Superpowers](https://github.com/obra/superpowers) adds TDD discipline and subagent code review. Each is powerful in isolation.

But none of them connect the dots from start to finish. You grill requirements, then what? You write specs, then what? You finish coding — who reviews it? You archive the change, but your README still describes last year's architecture.

Superpowers comes closest to a full workflow, but it's heavy — it brings its own planning system, worktree management, and assumes you buy into its entire methodology. Sometimes you just want OpenSpec's spec-driven flow with a grilling phase upfront and a code review at the end, without adopting a whole framework.

**aispecflow** fills that gap. It stitches together the best ideas from these tools into a single, lightweight plugin:

- Grill-style requirements interviewing (inspired by [Grill Me](https://github.com/mattpocock/skills) )
- OpenSpec's spec-driven artifacts (proposal → specs → design → tasks, inspired by [OpenSpec](https://github.com/Fission-AI/openspec))
- TDD implementation discipline (inspired by [Superpowers](https://github.com/obra/superpowers) )
- Independent subagent code review
- Documentation sync after every change

No custom planning system, no lock-in. Each skill works on its own. Use them together for the full flow, or pick the ones you need. Works equally well on existing codebases — run terrain-scan to understand what you have — or greenfield projects starting from scratch.

## Skills

| # | Skill | Phase | What it does |
|---|-------|-------|-------------|
| 0 | `terrain-scan` | Discover | Scan codebase → `docs/project-overview.md` |
| 1 | `seed-grill` | Grill | Relentless requirements interview → `docs/requirements.md`, `CONTEXT.md`, ADRs |
| 2 | `bloom-spec` | Spec | OpenSpec propose → proposal, specs, design, tasks |
| 3 | `grow-apply` | Apply | Implement tasks with TDD or straight (user choice) + git checkpoint commits + optional task-level review |
| 4 | `prune-review` | Review | Three-level review: task / change / project via independent subagent |
| 5 | `harvest-archive` | Archive | Sync specs, archive change, optional requirements archival |
| 6 | `renew-docs` | Refresh | Keep README + project-overview current (change-based or full refresh) |
| O | `axl-dev-flow` | Orchestrator | Full lifecycle: discover → grill → spec → apply → review → archive → refresh |

Every skill can be invoked independently. They share state through convention paths — no skill-to-skill RPC, no hidden coupling.

The skill names follow a growing lifecycle metaphor — from terrain to seed to bloom to growth to pruning to harvest to renewal:

```
terrain-scan → seed-grill → bloom-spec → grow-apply → prune-review → harvest-archive → renew-docs
```

Each phase feeds the next, like seasons in a garden. But unlike a real garden, you can jump into any season whenever you need it.

## Full Flow

```
┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
│ DISCOVER │──▶│  GRILL   │──▶│   SPEC   │──▶│  APPLY   │──▶│  REVIEW  │──▶│ ARCHIVE  │──▶│ REFRESH  │
│terrain-scan│   │seed-grill│   │bloom-spec│  │grow-apply │   │prune-review│  │archive-  │   │refresh-  │
└──────────┘   └──────────┘   └──────────┘   └──────────┘   └──────────┘   │change    │   │docs      │
      │              │              │              │              │         └──────────┘   └──────────┘
      ▼              ▼              ▼              ▼              ▼              │              │
project-       requirements   proposal +      code +        review           change        README +
overview.md    .md            specs +         tests         report          archived      project-
              CONTEXT.md      design +                        (+ reqs       (specs         overview.md
              ADRs            tasks                            optional)     synced)       updated
```

Run `/axl-dev-flow` for the full pipeline, or use any skill on its own.

## Project Structure After Use

```
your-project/
├── openspec/                          # OpenSpec spec-driven artifacts
│   ├── specs/                         # Main specs (synced after archive)
│   │   └── user-auth/
│   │       └── spec.md
│   └── changes/
│       ├── add-user-auth/             # Active change (before archive)
│       │   ├── proposal.md            # What & why
│       │   ├── specs/
│       │   │   └── user-auth/
│       │   │       └── spec.md        # Delta spec (requirements + scenarios)
│       │   ├── design.md              # Technical decisions
│       │   └── tasks.md               # Implementation checklist
│       └── archive/                   # Archived changes
│           └── 2026-07-05-add-user-auth/
│               ├── proposal.md
│               ├── specs/
│               ├── design.md
│               ├── tasks.md
│               └── requirements.md    # (if user chose to archive it)
├── docs/
│   ├── project-overview.md            # Generated by terrain-scan, refreshed by renew-docs
│   ├── requirements.md                # Generated by seed-grill
│   ├── adr/                           # Architectural Decision Records
│   │   └── 0001-use-postgres-for-sessions.md
│   └── .archive/                      # Backup files before modification
│       └── README.md.20260705-143000.bak
├── CONTEXT.md                         # Domain glossary (ubiquitous language)
├── README.md                          # Project readme (refreshed by renew-docs)
└── src/                               # Your actual code
```

## Installation

### Claude Code

```bash
claude plugin marketplace add comaaxl/aispecflow
claude plugin install aispecflow@aispecflow
```

### Codex

```bash
codex plugin marketplace add comaaxl/aispecflow
codex plugin add aispecflow@aispecflow
```

## Prerequisites

- [OpenSpec CLI](https://github.com/Fission-AI/openspec): `npm install -g @fission-ai/openspec`
- Git: required for task-level and change-level review ranges (grow-apply records a base commit and per-task checkpoints). Project-level review works without git. Without git, grow-apply still implements tasks, but task-level review is unavailable and change-level review degrades to the whole working tree.

## Optional: Worktree for isolated changes

This plugin does not manage git worktrees itself, but it is fully transparent to
them. If you prefer to isolate a change from your main branch, you can create a
worktree yourself and run the whole flow inside it:

```bash
git worktree add ../myapp-my-change -b my-change
cd ../myapp-my-change
# Run /axl-dev-flow (or /grow-apply -> /prune-review -> /harvest-archive) here.
# All git commands act on the my-change branch because they are relative to
# the current directory. The change-level review range (base..HEAD) is simply
# "from the worktree's branch point to now" - clean.
# When done, merge back to main (merge, not rebase, by default):
cd - && git merge my-change
git worktree remove ../myapp-my-change
```

Commit before merging - uncommitted changes in the worktree are not carried
over by `git merge` and are lost if you remove the worktree.

## License

MIT
