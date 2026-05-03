---
name: completion-audit
description: Use before shipping anything — run the completion checklist for your task type, triage failures by severity (BLOCKER / HIGH / LOW), and deliver a clear PASS / FAIL / CONDITIONAL PASS verdict.
---

# Completion Audit

## The Law
**Every task ships with a verdict, not reassurance. Run the checklist, score each item, triage by severity, and state the next action clearly. "It looks done" is not a verdict.**

## When to Use
- Before opening a pull request or merging to main
- A feature is built and you need to verify completeness
- A bug fix is implemented and you need sign-off
- Documentation or configuration changes are ready to ship
- **Never skip when:** anything user-facing or production-touching is about to merge — a FAIL verdict now prevents rollbacks later

## Process

### Phase 1: Classify the Task

Identify which task type applies. This determines which checklist runs.

| Type | Trigger |
|---|---|
| **Bug fix** | "fix", "broken", "error", "not working" |
| **Feature** | "add", "implement", "build", "create" |
| **Refactor** | "clean", "refactor", "restructure", "simplify" |
| **Docs / config** | "document", "update README", "configure", "env" |

If ambiguous, state the assumed type before proceeding.

### Phase 2: Run the Checklist

#### Bug Fix Checklist
- [ ] The specific error described no longer occurs
- [ ] The fix targets the root cause, not a symptom
- [ ] No new error surfaces on the happy path
- [ ] Edge cases that triggered the bug are covered
- [ ] The fix does not break adjacent functionality
- [ ] A test exists or was added that would catch a regression

#### Feature Checklist
- [ ] Every requirement stated in the task is implemented
- [ ] The feature works on the golden path without error
- [ ] Empty/null/zero inputs are handled
- [ ] Error states surface a useful message, not a crash
- [ ] The UI or API surface is consistent with existing conventions
- [ ] Documentation or types reflect the new capability

#### Refactor Checklist
- [ ] All existing tests still pass
- [ ] Behaviour is identical — no logic was changed
- [ ] The refactored code is shorter or clearer than the original
- [ ] No new `TODO` or dead code was introduced
- [ ] Public interfaces (signatures, exports) are unchanged

#### Docs / Config Checklist
- [ ] The change is accurate as of the current codebase state
- [ ] No outdated references remain in the changed section
- [ ] The format matches the surrounding file style
- [ ] Sensitive values (keys, passwords) are not hardcoded

### Phase 3: Score Each Unchecked Item

For every unchecked item, assign a severity:

| Severity | Meaning |
|---|---|
| **BLOCKER** | Ships broken without this — do not proceed |
| **HIGH** | Likely to cause issues in production or review |
| **LOW** | Worth fixing but will not break anything |

### Phase 4: Deliver the Verdict

Output in this exact format:

```
VERDICT: PASS | FAIL | CONDITIONAL PASS

Checked: X / Y items

BLOCKERS: <none, or list each with one line of detail>
HIGH:     <none, or list each with one line of detail>
LOW:      <none, or list each>

NEXT ACTION: <one sentence — what must happen before this ships, or "clear to ship">
```

**Verdict meanings:**
- **PASS** — all checklist items met, no blockers, no highs
- **CONDITIONAL PASS** — no blockers, ≤2 low issues noted
- **FAIL** — any blocker, or ≥2 highs

## Red Flags — Stop Immediately

- You are awarding a PASS to avoid friction — do not do this
- A checklist item is marked checked but you have not verified it — verify or uncheck it
- You are skipping the checklist because the task "looks done" — the checklist exists because looks are deceiving
- You are explaining what was done instead of evaluating whether the stated outcome was reached — only verdict matters

## Common Rationalizations

| Excuse | Why It's Wrong |
|--------|---|
| "The tests pass, so it must be done" | Tests are snapshots. They prove the code works on the cases you thought of. The checklist catches what tests miss. |
| "I've done this 100 times, I can skip the checklist" | Every task is different. Checklists are fast insurance. Skip the checklist once and you ship a problem that the checklist would have caught. |
| "The BLOCKER is not actually blocking, it's just a detail" | If you would not ship without it, it is blocking. If you would ship, do not call it a blocker. Call it what it is. |
| "I'll fix the HIGH issues after merging" | Post-merge fixes are always more expensive and have higher risk. Fix now or bump to CONDITIONAL PASS + future task. |
| "The user will discover the missing feature, that's okay" | Features you describe must be implemented before shipping. If a feature is not in scope, remove it from the task description. |

## Quick Reference

| Phase | Core Action | Done When |
|-------|---|---|
| **Classify** | Identify task type (bug / feature / refactor / docs) | Type is stated with reasoning |
| **Checklist** | Run all items for the task type | Every item is checked or flagged |
| **Score** | Assign BLOCKER / HIGH / LOW to unchecked items | Every gap has a severity |
| **Verdict** | State PASS / FAIL / CONDITIONAL PASS | Next action is explicit, verdict is final |
