---
name: receive-review
description: Use when processing incoming code review feedback — requires technical evaluation of each point before acting, not reflexive agreement or blanket implementation
---

# Receive Review

## The Law
**Every review comment must be verified against the actual codebase before a single line of implementation changes — understanding precedes action, and correctness precedes politeness.**

## When to Use
- Code review feedback arrives from any source (human, automated reviewer, external contributor)
- A batch of review items is listed and some are unclear
- **Never skip when:** feedback conflicts with an earlier architectural decision — that conflict must surface to the user before anything is touched

## Process

### Phase 1: Read the Full Feedback

1. Read every item before reacting to any of them.
2. Do not begin implementing while still reading.
3. Group items mentally: blocking, clarification needed, straightforward.

### Phase 2: Clarify Before Acting

If any item is unclear, stop here and ask — do not implement the parts you understood and leave the unclear ones for later.

```
Understand items 1, 2, 4. Need clarification on 3 and 5 before proceeding.
```

Partial implementation based on partial understanding produces incorrect results because related items often interact.

### Phase 3: Evaluate Each Item Technically

For each item, work through these checks before writing any code:

1. Is the suggestion technically correct for this specific codebase and runtime?
2. Would it break any existing functionality or tests?
3. Is there a reason the current implementation exists (compatibility, prior decision)?
4. Does the reviewer have the full context needed to make this call?
5. Does the change add something the codebase actually uses (YAGNI check)?

YAGNI check example:
```bash
# Check whether the suggested feature is actually called anywhere
grep -r "suggested_function_name" . --include="*.py"
grep -r "suggestedFunctionName" . --include="*.ts"
```

If the search comes back empty, raise it before building it.

### Phase 4: Triage and Order the Work

Once all items are understood and evaluated, sequence them:

1. Blocking issues — crashes, data corruption, security holes
2. Simple corrections — typos, wrong imports, off-by-one
3. Structural changes — refactoring, logic rewrites

### Phase 5: Implement One Item at a Time

1. Fix one item.
2. Run the relevant tests.
3. Confirm no regressions.
4. Move to the next item.

TypeScript example:
```typescript
// After each fix, run the specific test file
// npx jest src/auth/auth.service.spec.ts

// Then run the full suite before moving on
// npx jest
```

Python example:
```python
# After each fix, run the targeted test
# pytest tests/test_auth.py -v

# Then run the full suite
# pytest --tb=short
```

### Phase 6: Respond to the Reviewer

When an item is correct and fixed:
```
Fixed. <one sentence describing what changed and where>
```
or simply push the fix — the commit message serves as the response.

When pushing back on an item:
```
<Item N>: <technical reason this does not apply here>. <evidence: test name, line reference, or compatibility note>.
```

If you pushed back and the reviewer was right:
```
Verified — you were correct. My initial read was wrong because <specific reason>. Fixed in <location>.
```

No apologies, no gratitude expressions — the fix is the communication.

## Red Flags — Stop Immediately

- About to write "You're absolutely right!" or any equivalent praise before verifying the point
- Implementing a batch of items before clarifying everything in the batch
- Moving on to the next item without running tests on the current fix
- A reviewer suggestion conflicts with a prior architectural decision and the user has not been informed
- Adding a feature the codebase does not call anywhere (YAGNI violation)

## Common Rationalizations

| Excuse | Why It's Wrong |
|--------|----------------|
| "The reviewer must know best" | Reviewers lack full context; your job is to verify, not defer |
| "I'll ask about the unclear items after implementing the clear ones" | Related items interact; partial implementation with gaps creates inconsistency |
| "Pushback will seem defensive" | Technical correctness is not defensiveness; silence is not professionalism |
| "I'll skip the tests, it's a small fix" | Small fixes introduce regressions just as often as large ones |
| "The reviewer is external, so they are probably right" | External reviewers are more likely to lack codebase context, not less |

## Quick Reference

| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1 — Read | Consume all feedback | Every item seen before any reaction |
| 2 — Clarify | Ask about anything unclear | No ambiguous items remain |
| 3 — Evaluate | Technical check for each item | Applicability and correctness confirmed |
| 4 — Triage | Sequence by severity | Implementation order established |
| 5 — Implement | One item, test, repeat | All items addressed, tests green |
| 6 — Respond | State what changed or why you pushed back | Reviewer has actionable response |
