---
name: test-smells
description: Use when reviewing or writing tests — catch over-mocking, implementation coupling, and fragile assertions before they embed into the suite
---

# Test Smell Check

## The Law
**Tests must verify observable behaviour, not internal wiring. A test suite that passes after a correct refactor is trustworthy; one that breaks is testing the wrong thing.**

## When to Use
- Before merging any PR that adds or modifies tests
- When a test suite slows down or becomes brittle after refactoring
- When mock counts in a test file exceed the number of assertions
- **Never skip when:** a test is already failing for unclear reasons, or the suite feels fragile but nobody can say why

## Process

### Phase 1: Identify the Smell Category
1. Read each test and classify it against the smell list below.
2. Note the file, test name, and smell type — do not fix yet.
3. Group findings by severity: blocks merge, should fix, nice to fix.

**Smell catalogue:**

| Smell | Signature |
|-------|-----------|
| Mock overkill | More `mock`/`patch`/`jest.fn()` calls than assertions |
| Implementation coupling | Test breaks when internal variable names or private methods change |
| Assertion-free test | Test body has no `assert` / `expect` — relies on "no exception = pass" |
| Magic literals | Hardcoded numbers or strings with no explanation of their significance |
| Excessive setup | `setUp` or `beforeEach` longer than the test body itself |
| Duplicate coverage | Two tests exercise exactly the same branch with the same inputs |
| Vague name | Name does not describe the scenario being tested (`test_works`, `test1`) |
| Testing the mock | Assertions are only on mock call counts, not on the system under test |

### Phase 2: Assess Each Smell
1. For each flagged test, answer: "If production behaviour changes correctly, will this test break?"
   - Yes → the test is testing implementation, not behaviour.
   - No → the smell may be cosmetic; lower priority.
2. For each flagged mock: "Can the real dependency be used in a unit test without I/O?"
   - Yes → remove the mock; use the real thing.
   - No → keep the mock but assert on the output, not on call counts.

### Phase 3: Rewrite Smelly Tests
1. Fix one test at a time; run the suite after each fix.
2. Rewriting order: assertion-free first (highest risk), then mock overkill, then cosmetic.

**Before — testing the mock (Python)**
```python
def test_send_notification(mock_email):
    service = NotificationService(mock_email)
    service.notify(user_id=42, message="hello")
    mock_email.send.assert_called_once_with(to="user@example.com", body="hello")
```
This breaks if the implementation switches from `send` to `deliver` even if the email arrives.

**After — testing observable behaviour (Python)**
```python
def test_notify_delivers_message_to_user(fake_email_backend):
    service = NotificationService(fake_email_backend)
    service.notify(user_id=42, message="hello")
    assert fake_email_backend.sent_messages == [{"to": "user@example.com", "body": "hello"}]
```
Uses a fake (not a mock) and asserts on outcomes, not on method names.

**Before — implementation coupling (TypeScript)**
```typescript
test('parses config', () => {
  const parser = new ConfigParser();
  parser._tokenize('key=value');          // private method call
  expect(parser._tokens).toEqual([...]);  // private state
});
```

**After — behaviour only (TypeScript)**
```typescript
test('returns parsed key-value pairs from a valid config string', () => {
  const parser = new ConfigParser();
  expect(parser.parse('key=value')).toEqual({ key: 'value' });
});
```

### Phase 4: Report
1. List every smell found with file and line number.
2. Mark each as: fixed, deferred (reason), or accepted (reason).
3. If any assertion-free or mock-only tests remain unfixed, escalate — do not mark the review complete.

## Red Flags — Stop Immediately
- A test that cannot fail under any plausible production code change
- A test that imports private symbols or accesses `._private` attributes
- `assert True` or `expect(true).toBe(true)` — the test is a placeholder, not a guard
- Setup that stubs every method on the class under test (nothing real is being tested)
- A test named after a ticket number with no description of what it verifies

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "The mock proves the integration point is called" | Call-count assertions break on rename; test the outcome instead |
| "Private method testing catches bugs faster" | It catches refactoring, not bugs; test the public contract |
| "Removing the mock will make it an integration test" | Use a fake or in-memory substitute — unit tests do not require mocks |
| "This test is fine, it runs green" | Green does not mean correct; a test that can never fail provides no safety |
| "Fixing tests takes longer than the feature" | Smelly tests cost more in maintenance than they save in coverage |
| "We'll clean them up later" | Later never comes; smells compound with each new test added on top |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| Identify | Classify each test against the smell catalogue | Every test is labelled or cleared |
| Assess | Determine if smell is behavioural or cosmetic | Severity assigned to each finding |
| Rewrite | Fix assertion-free and mock-only tests first | Suite passes; no smell in the critical tier |
| Report | Document findings and resolution status | Nothing in the critical tier is deferred without a written reason |
