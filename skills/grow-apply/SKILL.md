---
name: grow-apply
description: Implement tasks from an OpenSpec change using test-driven development. Every task follows RED-GREEN-REFACTOR. Use when ready to implement a spec'd change.
---

# Apply — TDD Implementation

Implement tasks from an OpenSpec change. **Outer loop**: iterate through tasks.md. **Inner loop**: RED → GREEN → REFACTOR for every production code change.

## Iron Law

```
NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST.
```

If you wrote code before a test, delete the code. Start over. No exceptions.

## Pre-flight

### Step 1: Select the change

If a name is provided, use it. Otherwise:
- Run `openspec list --json` to get available changes
- If only one active change exists, auto-select it
- If multiple, ask the user to choose

Announce: "Implementing change: `<name>`"

### Step 2: Load context

```bash
openspec status --change "<name>" --json
openspec instructions apply --change "<name>" --json
```

Read all context files listed in the apply instructions output.

### Step 3: Choose implementation mode

Before starting, ask the user:

> "How would you like to implement these tasks?
> 1. **With TDD** (recommended) — write failing test first, then minimal code, then refactor. Each task follows RED → GREEN → REFACTOR.
> 2. **Straight implementation** — make code changes directly. Write tests after if needed."

If the user picks **TDD**: follow "The TDD Implementation Loop" below.
If the user picks **Straight**: for each task, make code changes, verify they work, mark complete, move on. Skip the RED-GREEN-REFACTOR inner loop.

### Step 4: Show current state

Display:
- Schema being used
- Tasks progress: "N/M tasks complete"
- Remaining tasks overview

## The TDD Implementation Loop

### Outer Loop: Iterate tasks

For each pending task in `tasks.md`:

```
┌─────────────────────────────────────────────┐
│  Task X.Y: <description>                    │
│                                             │
│  ┌─ Inner Loop: TDD ─────────────────────┐  │
│  │  RED:    write failing test            │  │
│  │  VERIFY: watch it fail correctly       │  │
│  │  GREEN:  write minimal code to pass    │  │
│  │  VERIFY: watch test + all others pass  │  │
│  │  REFACTOR: clean up, keep green        │  │
│  │  REPEAT: next test for this task       │  │
│  └────────────────────────────────────────┘  │
│                                             │
│  ✓ Mark task complete: `- [x] X.Y ...`     │
└─────────────────────────────────────────────┘
```

### Inner Loop: TDD Discipline

**RED — Write a failing test**
- One test, one behavior
- Clear name describing what it verifies
- Use real code, mock only at system boundaries (external APIs, databases, time)
- The test must verify observable behavior through public interfaces

**VERIFY RED — Watch it fail**
- Run the specific test: confirm it FAILS
- Confirm failure is because the feature is missing, not a typo
- If it passes, the test is wrong — fix the test
- **Never skip this step**

**GREEN — Write minimal code**
- Only enough code to make THIS test pass
- Don't add features, abstractions, or "future-proofing"
- YAGNI: no speculative parameters, branches, or flexibility the test doesn't require
- Don't refactor other code

**VERIFY GREEN — Watch it pass**
- Run the specific test: confirm it PASSES
- Run the full test suite: confirm no regressions
- Output must be clean — no errors, no warnings

**REFACTOR — Clean up**
- Only after all tests pass
- Extract duplication, improve names, extract helpers
- DRY: eliminate repeated logic, but stop at extraction the tests justify - don't abstract speculatively
- Keep tests green after each refactor step
- Don't add behavior during refactoring

**REPEAT** — Next test for the same task until the task is complete.

### Task Completion

After all tests for a task pass:
1. Mark the task complete in `tasks.md`: replace the specific `- [ ] X.Y ...` line with `- [x] X.Y ...`. **One line at a time. Never use global regex or batch sed across the entire file.**
2. Show progress: "Task X.Y complete (N/M done)"
3. Move to next task

### Pause Conditions

Stop and ask if:
- A task is unclear or ambiguous
- Implementation reveals a design issue (suggest updating artifacts)
- An error or blocker occurs that can't be resolved
- User interrupts

## Straight Implementation Mode

When the user chooses straight implementation:

For each pending task:
- Make the code changes required
- Keep changes minimal and focused
- YAGNI: only what the task requires - no speculative features or premature abstraction
- Verify the change works
- Mark task complete: replace the specific `- [ ] X.Y ...` line with `- [x] X.Y ...` — **one line at a time, no batch regex**
- Continue to next task

## What to Test

**Good tests** (write these):
```python
# Tests observable behavior through public interface
def test_user_can_authenticate_with_valid_credentials():
    result = authenticate("user@example.com", "correct-password")
    assert result.success is True
    assert result.token is not None

# Clear name, tests real behavior, one logical assertion
def test_authenticate_rejects_expired_token():
    expired_token = create_token(expires_in=-3600)
    result = validate_token(expired_token)
    assert result.valid is False
```

**Bad tests** (avoid these):
```python
# Tests implementation details — mocking internal collaborators
def test_authenticate_calls_password_hasher():
    mock_hasher = Mock(PasswordHasher)
    authenticate("user", "pass")
    assert mock_hasher.verify.called  # BAD: testing HOW, not WHAT
```

## When to Mock

Mock at **system boundaries only**:
- External APIs (payment, email, third-party services)
- Databases (prefer test database when possible)
- Time/randomness
- File system (when practical)

**Never mock** your own classes, modules, or internal collaborators.

## Completion

```
## Implementation Complete

**Change:** <name>
**Progress:** N/N tasks complete ✓

### Completed
- [x] Task 1: ...
- [x] Task 2: ...
...

All tasks complete! Prompt the user: "Implementation done. Run `/prune-review` to code-review before archiving."
```

## Guardrails

- **Delete code written before tests.** Iron Law. No exceptions (in TDD mode).
- **One test at a time.** Don't write multiple tests before implementing (in TDD mode).
- **Verify RED every time.** Never skip verification (in TDD mode).
- **Mark tasks one at a time.** Replace the exact line, never batch regex across the whole file.
- **Pause on ambiguity.** Don't guess — ask.
- **Communicate in the user's language.** Match the language the user writes in throughout the session. Never mix languages in a single response.
