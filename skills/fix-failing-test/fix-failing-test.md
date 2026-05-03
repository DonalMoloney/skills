---
name: fix-failing-test
description: Use when a test fails — diagnose the failure cause (not the implementation), never delete the test, fix the code to pass it, verify the test actually catches the bug.
---

# Fix Failing Test

## The Law
**Never delete a failing test. If a test fails, the code is wrong, not the test. Find what the test expects and fix the code to match.**

## When to Use
- A test suite is failing
- A specific test is broken
- Code changes broke an existing test
- You inherited a codebase with failing tests
- **Never skip when:** tests are failing on main/master — fix them immediately

## Process

### Phase 1: Understand the Failure
1. Run the failing test in isolation:
   ```bash
   pytest test_file.py::test_name -v
   ```
2. Read the failure message completely:
   - What was expected?
   - What actually happened?
   - What is the assertion that failed?
3. Never skip or ignore the output

### Phase 2: Classify the Failure
1. **Test is wrong:** Test logic is incorrect (rare)
2. **Implementation is wrong:** Code doesn't meet the test's expectation (common)
3. **Test is too strict:** Implementation is correct but test is fragile (sometimes)
4. **Test setup is wrong:** Mocks or fixtures are incorrect (sometimes)

Most failures are type 2 (implementation is wrong). Start there.

### Phase 3: Fix the Code
1. Find the code that the test is testing
2. Understand what the test expects
3. Modify the code to match the expectation
4. Verify the fix is minimal (one change, not rewrites)

### Phase 4: Run the Test Again
1. Run the failing test:
   ```bash
   pytest test_file.py::test_name -v
   ```
2. Verify it passes now
3. If it still fails, the fix was incomplete — go back to Phase 3

### Phase 5: Run the Full Test Suite
1. Make sure your fix didn't break other tests:
   ```bash
   pytest
   ```
2. If other tests broke, your fix was too broad — narrow it
3. All tests should pass before you're done

### Phase 6: Verify the Test Catches the Bug
1. Temporarily revert your fix
2. Run the test — it should fail again
3. Restore your fix
4. Run the test — it should pass

This confirms the test actually caught the bug, not a false positive.

## Red Flags — Stop Immediately

- You're deleting the failing test to make it pass — STOP and fix the code instead
- You don't understand what the test is testing — read it more carefully
- The test is checking an implementation detail, not a behaviour — the test might be too strict, but fix the code first
- Other tests are now failing — your fix was too broad
- You're modifying the test to pass instead of fixing the code — that's backwards

## Common Rationalizations

| Excuse | Why It's Wrong |
|--------|---|
| "This test is flaky, let's delete it" | Flaky tests reveal real bugs. Fix the flakiness, don't delete the test. |
| "The test is too strict" | Maybe. But first fix the code. Then decide if the test needs loosening. |
| "I don't have time to understand the test" | Understanding takes 5 minutes. Debugging the wrong problem takes hours. |
| "The test was working before my changes" | Then your changes broke something. Find and fix it. |
| "We'll fix this in the next sprint" | No. Broken tests on main compound into system-wide problems. Fix now. |

## Quick Reference

| Phase | Core Action | Done When |
|-------|---|---|
| **Understand** | Run test; read failure message completely | Failure is clear |
| **Classify** | Is it the test or the code? | Failure type is identified |
| **Fix** | Modify code to meet test expectation | Code matches test's intent |
| **Verify** | Re-run the failing test; it should pass | Test passes |
| **Full Suite** | Run all tests; confirm none broke | All tests pass |
| **Catch** | Revert fix, test fails; restore, test passes | Test actually catches the bug |
