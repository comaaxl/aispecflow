# Phase 5: Archive — Detailed Instructions

## Goal
Move the completed change to the archive, sync specs, optionally archive requirements.

## Trigger
Run `/harvest-archive`

## What Happens
1. Agent always lets user select the change (no auto-select)
2. Agent verifies artifacts done, tasks complete, review approved
3. Agent checks delta specs, shows comparison, offers Sync/Skip/Cancel
4. Agent checks docs/requirements.md, asks user: Archive/Archive-keep/Skip
5. Agent runs `openspec archive "<name>"`
6. Agent copies requirements.md if user chose to archive

## Completion Criterion
- Change is in archive directory
- Specs synced (or user chose to skip)
- Requirements handled per user's choice
