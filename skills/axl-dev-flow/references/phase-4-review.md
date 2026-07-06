# Phase 4: Review — Detailed Instructions

## Goal
Code review via an independent subagent. Catch issues before archiving. Prevents self-review bias.

## Trigger
Run `/prune-review`

## What Happens
1. Agent asks user what scope to review (5 options)
2. Agent gets git range or file list based on user's choice
3. Agent prepares context (description, plan/requirements)
4. Agent dispatches an independent code reviewer subagent with the reviewer prompt template
5. Evaluator verifies each issue before acting
6. Critical + Important issues fixed, re-reviewed if needed

## Completion Criterion
- All Critical issues fixed
- All Important issues fixed
- Reviewer verdict is "Approved" or "Approved with fixes"
