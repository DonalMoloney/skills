---
name: trace-bug
description: Use when a bug or failure is buried deep in a call stack and the direct error site is not the true origin — trace backward to find where bad data or bad state was introduced
---

# Trace Root Cause

## The Law
**Fix where the bad value originates, not where it first causes a crash. A fix at the symptom site leaves the source intact.**

## When to Use
- An exception is thrown inside a library or framework you did not write
- The same wrong value appears in multiple places across the codebase
- Fixing one error location immediately causes a different failure elsewhere
- Stack traces are long and the top frame is not meaningful on its own
- **Never skip when:** the error is deep in a third-party call stack, the same bug has reappeared after a previous fix, or you cannot explain why the failure happens at the line shown

## Process

### Phase 1: Read the Full Call Stack
1. Collect the complete traceback — do not truncate it
   ```python
   # Python — print the full exception chain including causes
   import traceback, sys

   try:
       run_operation()
   except Exception:
       traceback.print_exc()        # prints to stderr by default
       # or capture as a string
       tb = traceback.format_exc()
       print(tb, file=sys.stderr)
   ```
   ```bash
   # pytest — show full tracebacks without truncation
   pytest --tb=long tests/
   # or show each individual frame
   pytest --tb=native tests/
   ```
   ```typescript
   // Node — preserve the full stack in error handlers
   process.on('uncaughtException', (err) => {
     console.error(err.stack);
   });
   ```
2. Identify the bottom frame — this is the entry point that started the call chain
3. Identify the top frame — this is where execution stopped, often inside library code
4. Mark the first frame that is inside code you own; that is your starting investigation point

### Phase 2: Walk Backward Through the Chain
1. Start at the frame you own that is closest to the crash site
2. For each frame, answer: "What value or state was passed into this call, and where did it come from?"
3. Move one frame up (the caller) and repeat
4. Continue until you reach the frame where the bad value was first created or first mutated incorrectly
   ```python
   # Python — use pdb to walk the live call stack frame by frame
   import pdb; pdb.set_trace()
   # inside pdb: use `u` (up) and `d` (down) to move between frames
   # use `p variable_name` to inspect values at each level
   ```
   ```python
   # Python — inspect frames programmatically without a debugger
   import inspect
   frame = inspect.currentframe()
   while frame:
       local_vars = frame.f_locals
       print(f"{frame.f_code.co_filename}:{frame.f_lineno} — locals: {list(local_vars.keys())}")
       frame = frame.f_back
   ```
   ```typescript
   // Node — walk the call stack with Error capture
   const trace = {} as { stack?: string };
   Error.captureStackTrace(trace);
   console.log(trace.stack);
   ```
5. When you find the origin frame, note the exact line and the variable that carries the bad value

### Phase 3: Confirm the Origin
1. Add a targeted log or assertion at the suspected origin point
   ```python
   # Python — assert at the suspected origin to confirm
   assert isinstance(user_id, int) and user_id > 0, \
       f"Expected positive int user_id, got {user_id!r} (type {type(user_id).__name__})"
   ```
   ```python
   # Python — add a structured log before the suspicious call
   import logging
   log = logging.getLogger(__name__)
   log.debug("About to call process_order: order=%r, user_id=%r", order, user_id)
   ```
   ```typescript
   // TypeScript — narrow with a type guard at the suspected origin
   if (typeof userId !== 'number' || userId <= 0) {
     throw new Error(`Invalid userId at origin: ${JSON.stringify(userId)}`);
   }
   ```
2. Run the failing scenario and confirm the assertion or log fires at the expected location
3. If it does not fire, move one frame further up and repeat Phase 2 — the origin is earlier than expected

### Phase 4: Fix at the Source
1. Apply the fix at the origin point, not at any intermediate frame
2. Remove diagnostic logs and assertions added during investigation unless they have long-term value
3. Write a test that passes a bad value in at the origin point and confirms the correct rejection or correction
   ```python
   # Python — test that the origin now rejects bad input
   import pytest

   def test_process_order_rejects_invalid_user():
       with pytest.raises(ValueError, match="user_id must be a positive integer"):
           process_order(order=sample_order(), user_id=-1)
   ```
   ```typescript
   // TypeScript — confirm bad input is caught at the source
   it('rejects negative user IDs at the order entry point', () => {
     expect(() => processOrder({ ...sampleOrder, userId: -1 }))
       .toThrow('userId must be a positive integer');
   });
   ```
4. Verify no other test regressed; if a new failure appears in a different location, apply this entire process again from Phase 1

## Red Flags — Stop Immediately
- Fixing the crash site without checking where the bad value came from
- Stopping the trace at a library boundary and blaming the library without checking what you passed in
- Adding a guard at an intermediate frame instead of at the origin
- Assuming the top frame in the traceback is always the right fix site
- Copying the stack trace into a search engine and applying the first answer without tracing your own code

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "The error is in the library so it's not our bug" | Libraries fail on bad input; the bad input came from your code |
| "Adding a try/except here will stop the crash" | Swallowing the exception hides the origin and lets bad state propagate |
| "I'll add a null check at the crash site" | The null was created elsewhere; the check is a band-aid |
| "The stack is too long to trace fully" | Long stacks make origin tracing more important, not less |
| "It only fails in production so I can't debug it" | Add structured logging at key boundaries and redeploy; then read the logs |
| "The fix worked last time for a similar error" | Similar-looking errors can have different origins; trace each one independently |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1. Read Stack | Collect full traceback, find your first owned frame | Entry and crash frames identified |
| 2. Walk Backward | Trace callers up to the bad-value origin | Origin frame located |
| 3. Confirm Origin | Assert or log at suspected origin, reproduce failure | Assertion fires at the right location |
| 4. Fix at Source | Apply fix at origin, write a test, verify no regressions | Test green, symptom gone |
