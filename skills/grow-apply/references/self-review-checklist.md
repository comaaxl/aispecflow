# Self-Review Checklist

Used by the main conversation (NOT a subagent) after a task's tests pass and
before its checkpoint commit. This is a quick self-check for problems you
already know about - not a blind-spot finder (that is the subagent reviewer's
job, when enabled).

Run through this list mentally. Fix anything you find, inline, right now. Then
re-run the tests to confirm they still pass. Only then checkpoint commit.

## Checklist

- **Spec compliance** - Did this task do what its spec/task line required? Did
  it do MORE than required (YAGNI / scope creep - speculative parameters,
  unrequested flexibility, future-proofing)?
- **Obvious gaps** - Edge cases, error handling, empty input, failure paths:
  anything obviously missing that you know should be there?
- **Test quality** - Are the tests verifying real behavior (not mocking internal
  collaborators)? Is there an important case that should be tested but isn't?
- **Naming / readability** - Any unclear names, magic numbers, or code that
  would puzzle a reader?
- **Cut corners** - Any TODOs, temporary workarounds, or things you knowingly
  left rough and should clean up now?

## Rules

- Fix findings **inline** (in this conversation). Do not dispatch a fix agent -
  self-review fixes are always inline; `fix_mode` applies only to subagent
  review findings.
- After fixing, **re-run the tests** and confirm they still pass. If they
  don't, fix until green.
- Don't try to catch blind spots here. If the user enabled subagent review
  (per-task or final sweep), the independent reviewer catches those. Self-review
  is only for what you can already see.
- Keep it quick. If you find nothing on the list, move on - do not invent
  problems to look thorough.
