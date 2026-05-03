---
name: diagnose-bug
description: Use when debugging a mysterious bug — classify its type, build the root cause chain, determine fix strategy before implementing. Output a Bug Record with confidence level.
---

# Diagnose Bug

## The Law
**Do not fix the symptom without confirming the root cause. Treat the fix strategy as the output, not an implementation detail. Your diagnosis is not complete until you can answer: "If this root cause did not exist, would the symptom still occur?"**

## When to Use
- A bug report is vague or the cause is unclear
- Multiple failure modes could produce the same symptom
- The fix is not obvious from the error message
- A similar bug appeared before and you need to prevent regression
- **Never skip when:** the bug is in production or affects multiple users — guessing wastes time and risks bad fixes

## Process

### Phase 1: Define the Problem Precisely

Before any analysis, state:

```
PROBLEM STATEMENT
Expected: <what should happen>
Actual:   <what happens instead>
Trigger:  <minimum steps to reproduce>
Scope:    <one function | component-wide | system-wide>
```

If the trigger is unknown, state that explicitly — it changes the search strategy entirely.

### Phase 2: Classify the Bug Type

Assign a type immediately. Type determines where to look first.

| Type | Where to start | Search depth |
|---|---|---|
| **Logic** | Conditional branches, operators, off-by-one logic | Check control flow, variable state |
| **Runtime** | Null/undefined access, type coercion, unhandled exceptions | Check access patterns, type mismatches |
| **Integration** | API contract mismatch, wrong field names, auth headers, timeouts | Check request/response shape, handshakes |
| **Config** | Wrong env var, missing secret, environment-specific flag | Check env, secrets, feature flags |
| **Concurrency** | Race condition, shared mutable state, promise not awaited | Check threading, async ordering |
| **Data** | Invalid input, schema mismatch, stale cache, encoding | Check data shape, freshness |

State the type and your reasoning. If it could be multiple types, list all and rank by probability.

### Phase 3: Build the Root Cause Chain

Do not jump to a fix. Map the chain:

```
ROOT CAUSE CHAIN
Root:    <the underlying condition that made this possible>
Trigger: <the action that activated the root condition>
Symptom: <what the user sees or what the test catches>
```

Example:
```
Root:    userId is nullable but query does not guard against null
Trigger: unauthenticated request hits /profile
Symptom: TypeError: Cannot read properties of null (reading 'id')
```

If the chain requires more than three links, list them all — do not compress.

### Phase 4: State Your Assumptions

List every assumption you are making during analysis:

```
ASSUMPTIONS
- The error occurs in production, not just in tests
- The database schema has not changed recently
- The API response shape matches the type definition
```

If any assumption turns out wrong, restart from Phase 3 with correct facts.

### Phase 5: Fix Strategy (Before Code)

Before writing any code, state the fix in plain language:

```
FIX STRATEGY
Type:    <one-line fix description>
Touches: <files or functions that change>
Does not touch: <adjacent code that must remain unchanged>
Test to verify: <exact assertion or manual step that confirms the fix works>
```

Then implement the fix. Keep it minimal — change only what is necessary to break the root cause chain.

### Phase 6: Regression Check

After the fix, explicitly verify:

- [ ] The original symptom no longer occurs
- [ ] The code path that triggered the bug still exists but is now safe
- [ ] Adjacent functionality is not affected
- [ ] An existing test covers this path, or a new test was added

### Phase 7: Bug Record

Produce a compact bug record:

```
BUG RECORD
Type:       <classification from Phase 2>
Root cause: <one sentence>
Fix:        <one sentence>
Confidence: HIGH | MEDIUM
If MEDIUM:  <what you cannot verify without running the code>
Prevent:    <one thing that would prevent this class of bug in future>
```

## Red Flags — Stop Immediately

- You are fixing the symptom without understanding the root cause — stop and backtrack
- The classification is unclear — you need more evidence before proceeding
- The fix is larger than necessary to break the root cause chain — you are addressing side effects, not causes
- You cannot reproduce the bug — you cannot verify a fix
- High confidence flagged but you have not run the relevant code path — mark as MEDIUM instead

## Common Rationalizations

| Excuse | Why It's Wrong |
|--------|---|
| "We've seen this error before, I know what's wrong" | Familiarity breeds false positives. The previous bug and this one can have the same symptom but different roots. Classify and verify. |
| "The fix is obvious, let's skip the diagnosis" | Obvious fixes often treat the symptom. You end up with a patch that works "for now" and confuses the next person. |
| "I can't reproduce it, so it's probably gone" | Unreproducible bugs resurface at scale. Diagnosis cannot happen without reproduction — if you can't trigger it, the issue is not closed. |
| "The fix passes all tests, so it's safe" | Tests are snapshots. They prove the fix works on the cases you thought of. Regression checks catch wider impact. |
| "This is a one-off, we don't need to prevent it in future" | If it happened once, it can happen again. Prevention always pays off. |

## Quick Reference

| Phase | Core Action | Done When |
|-------|---|---|
| **Define** | State expected, actual, trigger, scope | Problem statement complete, trigger is known or assumed |
| **Classify** | Assign bug type; rank if multiple | Type is stated with reasoning |
| **Chain** | Map root → trigger → symptom | Full cause chain drawn, no gaps |
| **Assume** | List all assumptions | Assumptions are explicit and testable |
| **Strategy** | Write fix plan in plain language | Fix plan states what changes, what stays, how to verify |
| **Fix** | Implement minimal change | Change is complete and tested |
| **Verify** | Check regression, coverage | All verification checkboxes pass |
| **Record** | Bug record with confidence | Record includes type, root, fix, confidence, prevention strategy |
