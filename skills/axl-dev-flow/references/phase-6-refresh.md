# Phase 6: Refresh Docs — Detailed Instructions

## Goal
Update project documentation to reflect the current codebase.

## Trigger
Run `/renew-docs` (usable anytime, not just after archive)

## What Happens
1. Agent asks: change-based (using an archived change) or full refresh?
2. If change-based: user selects a change, agent reads its artifacts
3. If full refresh: agent re-scans the codebase
4. Agent diffs against README.md, docs/project-overview.md, CONTEXT.md
5. Agent proposes changes → waits for user consent
6. Agent backs up each file → applies approved changes

## Never Modify Without Consent
Always propose changes first. Never auto-edit documentation.

## Always Backup
```bash
cp <file> <file>.bak.$(date +%Y%m%d-%H%M%S)
```

## Completion Criterion
- Approved documentation changes applied
- User confirms docs are current
