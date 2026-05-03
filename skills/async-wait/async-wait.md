---
name: async-wait
description: Use when waiting for async conditions (API response ready, deployment healthy, server up). Never use sleep(). Use waitFor with timeout and polling instead.
---

# Async Wait

## The Law
**Never use sleep() in tests or production code. It burns time on every run, makes tests slow, and doesn't actually test the condition. Use condition-based waiting with timeout instead.**

## When to Use
- Waiting for a service to become healthy
- Waiting for an API to return a result
- Waiting for a file to exist or a state to change
- Waiting in tests for async operations to complete
- **Never skip when:** test timeouts are unpredictable — condition-based waiting fixes that

## Process

### Phase 1: Define the Condition
1. What are you actually waiting for?
   - File exists?
   - API response?
   - Port is listening?
   - State changed?
2. Write a check function that tests the condition:
   ```python
   def is_server_ready():
       try:
           response = requests.get("http://localhost:8000/health")
           return response.status_code == 200
       except:
           return False
   ```

### Phase 2: Set a Timeout
1. How long are you willing to wait?
   - Tests: 5-30 seconds
   - Deployments: 5-10 minutes
   - Startup checks: 30-60 seconds
2. Timeout should fail loudly, not silently

### Phase 3: Poll with Interval
1. How often should you check?
   - Tests: 100-500ms intervals (not too fast)
   - Production: 1-5 second intervals
   - Don't poll faster than necessary
2. Use backoff if polling expensive

### Phase 4: Implement Wait
1. **Python:**
   ```python
   import time
   def wait_for(condition, timeout=10, interval=0.1):
       start = time.time()
       while time.time() - start < timeout:
           if condition():
               return True
           time.sleep(interval)
       raise TimeoutError(f"Condition not met after {timeout}s")
   
   wait_for(is_server_ready, timeout=30)
   ```

2. **JavaScript:**
   ```javascript
   async function waitFor(condition, timeout=10000, interval=100) {
       const start = Date.now();
       while (Date.now() - start < timeout) {
           if (await condition()) return true;
           await new Promise(r => setTimeout(r, interval));
       }
       throw new Error(`Condition not met after ${timeout}ms`);
   }
   
   await waitFor(isServerReady, 30000);
   ```

### Phase 5: Add Clear Error Messages
1. When timeout occurs, include context:
   ```
   TimeoutError: Server did not respond with 200 after 30 seconds.
   Last check: {error details}
   ```

## Red Flags — Stop Immediately

- You used sleep() — replace with condition-based waiting
- Timeout is hardcoded to a long duration — make it configurable
- Error message is vague — add context about what failed
- Polling interval is very fast (< 50ms) — you're wasting CPU
- Condition function has side effects — it should only check state

## Common Rationalizations

| Excuse | Why It's Wrong |
|--------|---|
| "Sleep is simpler" | Sleep is wasteful. Condition-based waiting is faster and clearer. |
| "The condition is hard to check" | Then your code has a design problem. Make the condition observable. |
| "We don't need a timeout, it will eventually happen" | It won't. Network failures, crashes, and bugs exist. Always timeout. |
| "Polling is inefficient" | Polling is fine for 1-5 second waits. For longer waits, use webhooks or events. |

## Quick Reference

| Phase | Core Action | Done When |
|-------|---|---|
| **Define** | Write check function that tests condition | Condition is observable and testable |
| **Timeout** | Set realistic timeout for the context | Timeout is explicit and reasonable |
| **Interval** | Choose polling interval (100-5000ms) | Interval balances responsiveness and CPU |
| **Implement** | Code wait_for() loop with timeout and polling | Loop checks condition, waits, repeats, timeouts |
| **Error** | Fail with clear message and context | Failure explains what didn't happen |
