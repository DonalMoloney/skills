---
name: find-silent-failures
description: Invoke when reviewing error handling code, catch blocks, fallback logic, or any PR where errors could be swallowed without a trace.
---

# Find Silent Failures

## The Law
**Every error that occurs in production must be logged, surfaced, or propagated — swallowing an error is never a valid handling strategy.**

## When to Use
- Reviewing a PR that touches try/except or try/catch blocks
- After writing any code with fallback logic or default-on-error return values
- Pre-PR sanity check on any module that calls external services
- **Never skip when:** the diff contains `except:`, `catch (e)`, `.catch(`, or any `|| default` pattern — these are prime silent-failure locations

## Process

### Phase 1: Map All Error Handling Sites
1. Scan the diff for every location where an error can be caught or suppressed
   - `try/except` and `try/catch` blocks
   - `.catch()` promise handlers and `onerror` callbacks
   - Optional chaining (`?.`) and null-coalescing (`??`) that could mask failures
   - Return values like `null`, `[]`, `{}`, or `False` returned from an except branch
   - Retry loops — check what happens after all attempts are exhausted
2. List each site with its file path and line range before moving to analysis

### Phase 2: Interrogate Each Handler
For every site found in Phase 1, answer these questions in order:

1. **Is the error logged?**
   - Is severity appropriate — warning vs error vs critical?
   - Does the log carry enough context: operation name, relevant IDs, input state?
   - Will the log be actionable to an on-call engineer six months from now?

2. **Does the user know something went wrong?**
   - Is there a user-facing message, notification, or signal?
   - Is the message specific enough to distinguish this failure from others?
   - Does it tell the user what they can do next?

3. **Is the catch too broad?**
   - List every exception type that could be caught that the developer did NOT intend to handle
   - A bare `except:` or `catch (Exception e)` hides programming errors, network failures, and permission errors equally — each warrants different treatment

4. **Is the fallback justified?**
   - Does a product requirement explicitly call for this fallback?
   - Would a user be confused to see fallback behavior with no explanation?
   - Does the fallback hide a defect that should be fixed instead?

5. **Should the error propagate?**
   - Would a higher-level handler deal with this better?
   - Does catching here prevent proper resource cleanup?

### Phase 3: Check for Hidden-Failure Patterns
Look explicitly for each of these patterns — each is always a defect:

1. Empty except block — error disappears completely
   ```python
   try:
       result = fetch_data(user_id)
   except Exception:
       pass   # DEFECT: no log, no signal, no re-raise
   ```

2. Log-and-continue with a false success return
   ```python
   def send_notification(user: User) -> bool:
       try:
           notifier.deliver(user)
           return True
       except RequestException as e:
           logger.error("delivery failed: %s", e)
           return True   # DEFECT: caller believes success; it was not
   ```

3. Silent fallback to a default value
   ```python
   def load_config(path: str) -> Config:
       try:
           return Config.from_file(path)
       except FileNotFoundError:
           return Config()   # DEFECT: no log, no signal — broken config ships invisibly
   ```

4. Fallback to a mock or stub outside test code
   ```python
   try:
       client = PaymentGateway()
   except ConnectionError:
       client = FakePaymentGateway()   # DEFECT: production code using a test double
   ```

5. Broad catch masking unrelated failures (TypeScript contrast)
   ```typescript
   try {
     const data = JSON.parse(response.body);
     processData(data);
   } catch (e) {
     return null;   // DEFECT: TypeError, SyntaxError, and programmer mistakes are all silenced
   }
   ```

### Phase 4: Evaluate Error Messages
For every user-facing error string:

1. Is it written in terms the target user understands?
2. Does it name the operation that failed — not just "an error occurred"?
3. Does it provide a concrete next step ("Try again in a moment" or "Contact support with code X-42")?
4. Is it distinct enough to find in logs and distinguish from similar failures?

### Phase 5: Validate Against Project Standards
1. Open CLAUDE.md and check for project-specific logging functions, error ID registries, or Sentry conventions
2. Confirm error codes or IDs follow the project's naming scheme
3. Confirm empty catch blocks are absent — none are acceptable under any circumstance
4. Confirm test doubles (Mock, Stub, Fake) appear only in test files

## Red Flags — Stop Immediately
- `except:` or `except Exception:` with no re-raise and no log
- `catch (e) {}` — empty catch body in any language
- Function returning a success indicator from inside an except branch
- Production code that imports or instantiates a Mock, Stub, or Fake class
- Retry loop that silently returns a default when all attempts are exhausted
- Optional chaining (`?.`) used to silently skip an operation that should raise when absent
- `# TODO: handle error` inside a catch block — it means the error ships unhandled

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "Failing silently is fine for a non-critical feature" | Silent failures accumulate; the next failure is harder to diagnose because this one left no trace |
| "We log it upstream" | Upstream logs lose the local context — operation name, IDs, state — that makes the log useful |
| "The fallback is harmless" | A harmless fallback today becomes invisible wrong data six months from now |
| "Catching broadly keeps things simple" | Broad catches turn programming errors into mysterious state corruption |
| "It only fails in edge cases" | Edge cases become the norm under production load and appear at the worst possible moment |
| "The user doesn't need to know about this" | Users need to know — they are the ones who retry, report, and escalate |
| "We'll add proper handling in the next sprint" | Later never arrives; ship the handler now or ship the bug |
| "It's just test code" | Silent failures in tests produce false-green suites — worse than no tests at all |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1. Map | List every error-handling site | Full inventory with file paths |
| 2. Interrogate | Log? User signal? Catch scope? Fallback justified? Propagate? | Five questions answered per site |
| 3. Patterns | Scan for the five hidden-failure patterns | Each pattern explicitly checked |
| 4. Messages | Review every user-facing error string | Specific, actionable, searchable |
| 5. Standards | Cross-check CLAUDE.md conventions | No violations remain |
