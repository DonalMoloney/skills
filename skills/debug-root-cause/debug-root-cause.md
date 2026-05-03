---
name: debug-root-cause
description: Use when any bug, test failure, or unexpected behavior appears — before proposing any fix
---

# Debug Root Cause

## The Law
**Never propose a fix until you have identified the root cause. Treating symptoms is failure.**

## When to Use
- Any test failure, regardless of how obvious the fix looks
- Unexpected runtime behavior or incorrect output
- Build or integration failures
- Performance regressions
- Recurring bugs that keep coming back
- **Never skip when:** you're under time pressure, the fix "seems obvious," you've already tried two or more things, or you don't fully understand why the failure is happening

## Process

### Phase 1: Understand Before Touching
1. Read the full error message — do not skip warnings or stack traces
   - Note file paths, line numbers, error codes, and exception types
   - Stack traces point to the origin; read them bottom-up
2. Reproduce the failure consistently
   - Identify the exact steps or inputs that trigger it every time
   - If you cannot reproduce it reliably, gather more data before proceeding — do not guess
3. Check what recently changed
   - Run `git diff` and review recent commits
   - Check for new dependencies, config edits, or environment differences
4. Instrument component boundaries in multi-layer systems
   - For each layer (API → service → database, CI → build → signing), add diagnostic output:
     ```bash
     # Python: confirm env propagation at each layer
     import os, sys
     print(f"[layer-1] DB_URL={'SET' if os.getenv('DB_URL') else 'MISSING'}", flush=True)
     ```
     ```bash
     # TypeScript/shell: confirm at each boundary
     echo "=== Layer 1 env check ==="
     echo "DB_URL: ${DB_URL:+SET}${DB_URL:-MISSING}"
     ```
   - Run once to see where it breaks — then investigate that layer
5. Attach a debugger if the failure is inside a function
   ```python
   # Python — drop into pdb at the failing line
   import pdb; pdb.set_trace()
   # or run the whole file under pdb
   python -m pdb my_script.py
   ```
   ```bash
   # Run pytest and open pdb on the first failure
   pytest --pdb -x tests/test_payments.py
   ```
   ```typescript
   // Node — built-in inspector
   node --inspect-brk dist/server.js
   ```

### Phase 2: Find the Pattern
1. Locate working examples of similar code in the same codebase
2. Read the reference implementation completely — do not skim
3. List every difference between the working version and the broken one, however minor
4. Map out what the broken code depends on: config, environment, shared state, call order

### Phase 3: Form and Test a Hypothesis
1. Write a single, specific hypothesis: "I think X is failing because Y"
   - Vague hypotheses produce vague tests — be precise
2. Make the smallest possible change that would confirm or refute the hypothesis
   - One variable at a time; no bundled changes
3. Run the test and read the output in full
   ```python
   # Python — capture and inspect traceback programmatically
   import traceback
   try:
       result = suspect_function(input_value)
   except Exception:
       traceback.print_exc()   # prints full chain to stderr
   ```
4. If the hypothesis is wrong, form a new one — do not stack another fix on top
5. If you genuinely do not understand the mechanism, say so and research further rather than guessing

### Phase 4: Fix and Confirm
1. Write a failing test that reproduces the bug before touching production code
   ```python
   # Python — minimal reproduction as a pytest test
   def test_payment_fails_on_zero_amount():
       with pytest.raises(ValueError, match="amount must be positive"):
           process_payment(amount=0)
   ```
   ```typescript
   // TypeScript — minimal Jest reproduction
   it('rejects zero-amount payments', () => {
     expect(() => processPayment({ amount: 0 })).toThrow('amount must be positive');
   });
   ```
2. Apply one fix that addresses the root cause — not the symptom
3. Confirm the failing test now passes and no other tests regressed
4. If the fix does not work, stop — do not attempt a fourth fix without stepping back
   - Three failed fixes signal an architectural problem, not a missing tweak
   - Discuss with the team before making more changes

## Red Flags — Stop Immediately
- "Let me just try changing X and see what happens"
- Proposing fixes before completing Phase 1
- Making multiple changes at once to "narrow it down"
- Skipping the failing test and verifying manually instead
- "I don't fully understand it but this might work"
- Seeing the same bug surface in a different place after each fix
- Reaching for a fourth fix after three have already failed

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "It's a simple bug, no need for process" | Simple bugs have root causes too; the process is fast when the bug is genuinely simple |
| "We're in a hurry, skip the diagnosis" | Guessing takes longer than investigating — thrashing wastes more time than a methodical pass |
| "I'll write the test after I confirm the fix works" | Untested fixes drift; a test written first proves the fix actually addresses the right thing |
| "I'll change a few things at once to save time" | You cannot tell which change fixed it, and you may introduce a second bug |
| "I can see the problem — let me fix it" | Spotting a symptom is not the same as understanding the cause |
| "The reference is too long; I'll adapt the pattern" | Partial understanding of a pattern guarantees gaps that will surface later |
| "One more attempt" (after two or more failures) | Three failed fixes mean something structural is wrong — more patches compound the problem |
| "It only fails sometimes, so I'll ship and watch" | Intermittent bugs are reproducible with the right inputs; find them before users do |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1. Understand | Read errors, reproduce, instrument boundaries | You know what is failing and where |
| 2. Pattern | Find working examples, compare differences | You know how it should behave |
| 3. Hypothesis | State theory, test with one change | Hypothesis confirmed or replaced |
| 4. Fix | Write failing test, apply root-cause fix, verify | Test green, no regressions |
