---
name: verify-before-done
description: Use before marking work complete — verify the task works as specified through manual testing, automation checks, and edge cases. Output a verification checklist and sign-off.
---

# Verify Before Done

## The Law
**Do not mark a task complete without verifying it works. "The code looks good" is not verification. Test the golden path, test edge cases, test error states, then sign off.**

## When to Use
- A feature is built and ready to call done
- A bug fix is implemented and needs verification
- You are about to ask for code review
- **Never skip when:** anything user-facing or production is about to merge

## Process

### Phase 1: Define Success Criteria
1. Read the original task description
2. List what must work for this task to be DONE
3. Identify edge cases and error states

### Phase 2: Test the Golden Path
1. Follow the happy path steps manually
2. Verify each step works without error
3. Record the result:
   ```
   GOLDEN PATH TEST
   Step 1: ... ✓
   Step 2: ... ✓
   Step 3: ... ✓
   ```

### Phase 3: Test Edge Cases
1. Test empty/null/zero inputs
2. Test boundary values (max, min, overflow)
3. Test unusual but valid inputs
4. Record failures as blockers

### Phase 4: Test Error States
1. What happens when the operation fails?
2. Is the error message useful?
3. Can the user recover?
4. Are there graceful fallbacks?

### Phase 5: Verify Automation
1. Do all tests pass?
   ```bash
   # Python
   pytest

   # TypeScript
   npm test
   ```
2. Do type checks pass?
   ```bash
   # Python
   mypy --strict .

   # TypeScript
   npx tsc --noEmit
   ```
3. Does linting pass?
   ```bash
   # Python
   ruff check .

   # TypeScript
   npm run lint
   ```

### Phase 6: Sign Off
1. Output a verification summary:
   ```
   VERIFICATION COMPLETE
   Golden path: ✓
   Edge cases: ✓
   Error handling: ✓
   Tests: ✓ (X/X passing)
   Types: ✓
   Lint: ✓
   
   VERDICT: READY FOR REVIEW
   ```

## Red Flags — Stop Immediately

- The golden path doesn't work — do not proceed
- Tests are failing — do not proceed
- Type errors exist — do not proceed
- Lint failures exist — do not proceed
- You didn't test a documented requirement — test it now
- Error handling is missing — add it and test it

## Common Rationalizations

| Excuse | Why It's Wrong |
|--------|---|
| "The tests pass, so it must work" | Tests are snapshots. They prove the code works on the cases you thought of. Manual testing catches what tests miss. |
| "I'll test it in code review" | Code review catches code issues, not functional issues. Test first, then review. |
| "This is a tiny change, I don't need to verify" | Tiny changes break things. Verify everything before considering it done. |
| "The error handling is obvious from the code" | Obvious error handling often has gaps. Test it and verify the message is useful. |
| "I tested the happy path, that's enough" | Happy path failures are rare. Failures happen in edge cases and error states. Test those. |

## Quick Reference

| Phase | Core Action | Done When |
|-------|---|---|
| **Define** | State success criteria and edge cases | Criteria are explicit and testable |
| **Golden Path** | Walk through happy path manually | All steps work without error |
| **Edge Cases** | Test boundaries, nulls, unusual inputs | Edge cases handled correctly |
| **Errors** | Test failure modes; verify error messages | Error handling is graceful and useful |
| **Automation** | Run tests, types, lint | All checks pass |
| **Sign Off** | Output verification checklist; state verdict | READY FOR REVIEW or blocker found |
