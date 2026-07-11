---
name: prune-review
description: Dispatch a code reviewer subagent to evaluate completed work against requirements, code quality, and architecture. Use after implementation is complete or at natural checkpoints.
---

# Review — Code Review

Dispatch an independent code reviewer subagent to evaluate the implementation against its requirements. The reviewer must be a separate agent with no access to your session context — this prevents self-review bias.

**Core principle**: Review early, review often. Catch issues before they cascade.

## When to Use

**Mandatory**:
- After completing all tasks in `/grow-apply`
- After completing a major feature
- Before running `/harvest-archive`

**Optional**:
- Between task groups (natural checkpoints)
- When stuck (fresh perspective)
- Before large refactors

## Platform Adaptation

Different agent platforms have different support for key capabilities (e.g., dispatching subagents). Before running this skill, check whether your harness has special instructions.

If your harness appears here, read its reference file for platform-specific setup:

- Codex: `references/codex-tools.md`

## Process

### Step 0: Ask the User What to Review

> "What scope should I review?
> 1. All uncommitted changes (working tree diff)
> 2. Latest commit only (git diff HEAD~1..HEAD)
> 3. Since branching off main (git diff origin/main..HEAD or merge-base)
> 4. Specific commit range (you tell me: base..head)
> 5. Specific files (you tell me which files)"

### Step 1: Determine review scope

Based on the user's choice, get the git range or file list.

### Step 2: Prepare context

Collect:
- **Description**: What was built (from `proposal.md` or ask user)
- **Requirements**: Completed tasks from `tasks.md` + `specs/`
- **Change name**: The OpenSpec change name, if applicable

### Step 3: Dispatch independent code reviewer

**CRITICAL**: The reviewer must be a separate subagent, not you acting as reviewer. Using the reviewer prompt template from `references/code-reviewer.md`. Fill placeholders: `{DESCRIPTION}`, `{PLAN_OR_REQUIREMENTS}`, `{BASE_SHA}`, `{HEAD_SHA}`.

The reviewer gets ONLY the template — never your session context.

The reviewer checks:
1. **Plan alignment**: Implementation matches requirements? Deviations justified?
2. **Code quality**: Separation of concerns, error handling, type safety, DRY
3. **Architecture**: Sound design, scalability, security, integration
4. **Testing**: Tests verify behavior (not mocks), edge cases covered, all passing
5. **Production readiness**: Migration strategy, backward compatibility, docs

### Step 4: Evaluate the review

When the reviewer returns:

```
Reviewer output:
  Strengths: ...
  Issues: Critical (N), Important (N), Minor (N)
  Assessment: Ready to merge? [Yes | No | With fixes]
```

**For each issue, verify before acting:**
- Is it technically correct for THIS codebase?
- Does it break existing functionality?
- Is there a reason for the current implementation?

**If reviewer is wrong**: Push back with technical reasoning.
**If reviewer is correct**: Fix Critical immediately, Important before archive, Minor note for later.

### Step 5: Act on feedback

```
Critical → Fix now, re-review
Important → Fix before archive
Minor → Note for future
```

After all Critical + Important resolved:

> "Code review complete. All Critical and Important issues resolved. Ready for `/harvest-archive`."

## Receiving Review Feedback (Discipline)

```
1. READ: Complete feedback without reacting
2. UNDERSTAND: Restate requirement in own words
3. VERIFY: Check against codebase reality
4. EVALUATE: Technically sound for THIS codebase?
5. RESPOND: Technical acknowledgment or reasoned pushback
6. IMPLEMENT: One item at a time, test each
```

**Never:**
- Performative agreement ("You're absolutely right!")
- Blind implementation (verify first)
- Argue with valid technical feedback

**Push back when:**
- Suggestion breaks existing functionality
- Reviewer lacks full context
- Violates YAGNI
- Technically incorrect for this stack

## Output

```
## Review Complete

**Change:** <name>
**Reviewer Verdict:** Approved / Approved with fixes / Needs work
**Scope reviewed:** <git range or file list>

### Resolved
- [x] Critical: <issue> → Fixed in <file>
- [x] Important: <issue> → Fixed in <file>

### Deferred (Minor)
- [ ] <issue> — noted for future

Prompt the user: "Review approved. Run `/harvest-archive` to archive this change."
```

## Guardrails

- **Must use a separate subagent, not self-review.** 
- **Ask the user what to review before determining git range.**
- **Handle missing git.** If git is not available, review the working tree directly.
- **Never skip review.**
- **Verify before implementing feedback.**
- **No performative agreement.**
- **Communicate in the user's language.** Match the language the user writes in throughout the session. Never mix languages in a single response.
