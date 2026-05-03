---
name: tdd-workflow
description: Use when implementing any feature or bug fix — write the failing test before a single line of production code
---

# TDD Workflow

## The Law
**No production code exists without a failing test that demanded it. Write code before the test? Delete it and start over.**

## When to Use
- Any new function, class, or module
- Any bug fix — the bug must become a failing test before the patch lands
- Any behaviour change, even a one-liner
- **Never skip when:** you think the change is "too simple to bother", you're already halfway through implementing, or you believe manual testing is equivalent

## Process

### Phase 1: RED — Write One Failing Test
1. Decide on a single, concrete behaviour to implement next.
2. Write the smallest possible test that describes that behaviour.
   - Name the test so a stranger knows what behaviour it guards.
   - Use real objects — reach for mocks only when external I/O is unavoidable.
3. Run the test suite and confirm the new test fails.
4. Read the failure message — it must say "feature missing", not "syntax error" or "import error".
   - Wrong failure message: fix the test, run again.
   - Test passes immediately: you are testing existing behaviour — fix the test.

**Python (pytest)**
```python
# test_retry.py
def test_retries_failed_operation_three_times():
    attempts = 0

    def flaky():
        nonlocal attempts
        attempts += 1
        if attempts < 3:
            raise ValueError("not yet")
        return "done"

    result = retry(flaky)

    assert result == "done"
    assert attempts == 3
```

**TypeScript (Jest)**
```typescript
test('retries a failing operation three times before succeeding', async () => {
  let attempts = 0;
  const flaky = () => {
    attempts++;
    if (attempts < 3) throw new Error('not yet');
    return Promise.resolve('done');
  };

  const result = await retry(flaky);

  expect(result).toBe('done');
  expect(attempts).toBe(3);
});
```

### Phase 2: GREEN — Minimal Passing Code
1. Write the smallest amount of production code that makes the failing test pass.
2. Do not add parameters, options, or logic the test does not require.
3. Do not refactor other code while in this phase.
4. Run the full test suite — the new test must pass and no previously passing test may fail.

**Python**
```python
def retry(fn, max_attempts=3):
    for attempt in range(max_attempts):
        try:
            return fn()
        except Exception:
            if attempt == max_attempts - 1:
                raise
```

**TypeScript**
```typescript
async function retry<T>(fn: () => Promise<T>, maxAttempts = 3): Promise<T> {
  for (let i = 0; i < maxAttempts; i++) {
    try {
      return await fn();
    } catch (e) {
      if (i === maxAttempts - 1) throw e;
    }
  }
  throw new Error('unreachable');
}
```

### Phase 3: REFACTOR — Clean Without Adding
1. Remove duplication introduced in the GREEN phase.
2. Improve names and extract helpers where the intent is unclear.
3. Run the full suite after every change — stay green throughout.
4. Add no new behaviour. If refactoring reveals a gap, write a new test in Phase 1.

### Phase 4: Repeat
1. Pick the next single behaviour.
2. Return to Phase 1.

## Why Order Matters
- A test written after production code answers "what does this code do?" — not "what should it do?"
- A test that never fails cannot prove it catches the bug it names.
- Implementation bias means post-hoc tests reflect the code you wrote, not the requirements you meant to meet.
- Watching a test fail is the only proof the test is actually wired to the right thing.

## Red Flags — Stop Immediately
- You wrote production code before any test existed
- The new test passed on the first run without any implementation
- You cannot explain why the test failed in Phase 1
- You are holding on to exploratory code as "reference" while writing tests
- You are writing multiple tests in one go before any implementation
- You are about to skip TDD "just for this one change"

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "This function is trivial" | Simple code still breaks; the test takes under a minute |
| "I'll add tests right after" | A test written after passing code proves the code compiles, nothing more |
| "Manual testing covered this" | Manual runs are not repeatable; they vanish the moment you move on |
| "Deleting hours of work is wasteful" | Sunk cost — unverified code is debt; rewriting with TDD costs one afternoon |
| "I'll keep it as reference and write tests first" | You'll adapt it; adapting existing code is test-after; delete means delete |
| "TDD is dogma — being pragmatic means adapting" | TDD finds bugs before commit and prevents regressions; skipping it is slower overall |
| "Hard to test so the test is the problem" | Hard to test signals tight coupling — the test is telling you to simplify the interface |
| "The existing codebase has no tests anyway" | Adding coverage to what you touch today improves the baseline; start with this change |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| RED | Write one test for one behaviour | Test fails with the expected message |
| GREEN | Write minimal production code | New test passes; full suite still passes |
| REFACTOR | Remove duplication and improve names | Suite stays green; no new behaviour added |
| REPEAT | Pick the next behaviour | Next RED test is written |
