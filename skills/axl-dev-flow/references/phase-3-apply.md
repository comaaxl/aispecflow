# Phase 3: Apply — Detailed Instructions

## Goal
Implement all tasks. Choice of TDD or straight implementation.

## Trigger
Run `/grow-apply`

## What Happens
1. Agent selects the change
2. Agent runs `openspec instructions apply` to get context
3. Agent asks user: TDD (RED→GREEN→REFACTOR) or straight implementation?
4. Agent implements tasks one at a time, marking each immediately
5. Task marking uses exact line replacement — no batch regex

## TDD Iron Law
NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST.

## Completion Criterion
- All tasks checked `[x]`
- All tests green

## Pause Conditions
- Task is unclear → ask
- Design issue revealed → suggest updating artifacts
- Error/blocker → report and wait
