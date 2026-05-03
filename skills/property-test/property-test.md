---
name: property-test
description: Use when a function processes a range of inputs — generate hundreds of random cases automatically with Hypothesis (Python) or fast-check (TypeScript)
---

# Property Test

## The Law
**Every function with a non-trivial input space must have at least one property test that the framework drives with generated data — hand-picked examples alone cannot cover the edge cases you haven't thought of yet.**

## When to Use
- Parsing, serialisation, or encoding functions where the input space is large
- Pure functions with mathematical invariants (sort stability, idempotency, round-trips)
- Validation logic where boundary conditions matter
- Any function where you have written more than three example-based tests for the same behaviour
- **Never skip when:** the function handles user-supplied strings, numbers, or collections — adversarial inputs will find the gaps faster than you will

## Process

### Phase 1: Identify the Property
1. Ask: "What must always be true about the output, regardless of which valid input I supply?"
2. Choose a property type:
   - **Round-trip**: encode then decode returns the original value
   - **Invariant**: output always satisfies a fixed condition (sorted, non-negative, valid JSON)
   - **Idempotency**: applying the function twice gives the same result as applying it once
   - **Oracle**: compare against a reference (slow but correct) implementation
   - **Metamorphic**: a known input transformation produces a predictable output transformation
3. Write the property in plain language before touching code.

### Phase 2: Write the Property Test

**Python — Hypothesis**
```python
from hypothesis import given, assume, settings
from hypothesis import strategies as st


@given(st.lists(st.integers(), min_size=1))
def test_sort_is_stable_and_preserves_elements(values):
    result = my_sort(values)

    assert sorted(result) == sorted(values)   # same elements
    assert result == sorted(result)            # ascending order
```

```python
# Round-trip property
@given(st.text())
def test_encode_decode_round_trips(original):
    assert decode(encode(original)) == original
```

```python
# Idempotency property
@given(st.text())
def test_normalise_is_idempotent(s):
    once = normalise(s)
    twice = normalise(once)
    assert once == twice
```

**TypeScript — fast-check**
```typescript
import * as fc from 'fast-check';

test('sort preserves all elements and produces ascending order', () => {
  fc.assert(
    fc.property(fc.array(fc.integer(), { minLength: 1 }), (values) => {
      const result = mySort(values);
      expect(result).toEqual([...values].sort((a, b) => a - b));
    })
  );
});
```

```typescript
// Round-trip property
test('encode then decode returns the original string', () => {
  fc.assert(
    fc.property(fc.string(), (original) => {
      expect(decode(encode(original))).toBe(original);
    })
  );
});
```

### Phase 3: Run and Interpret Failures
1. Run the property test — the framework will try hundreds of inputs automatically.
2. If the test fails, the framework will report the **shrunk minimal failing example**.
3. Do not treat the minimal example as the only case to fix — the property must hold for all inputs.
4. Fix the production code, then re-run to confirm the framework finds no further counterexample.

**Reading a Hypothesis failure**
```
Falsifying example: test_encode_decode_round_trips(
  original='\x00',
)
```
The framework found a null byte breaks the round-trip, then shrunk the input to the smallest reproducing case.

### Phase 4: Tune Generators and Constraints
1. Use `assume()` (Hypothesis) or `.filter()` (fast-check) to skip inputs that violate preconditions — do not silently swallow them.
2. Add `@settings(max_examples=500)` or `fc.assert(..., { numRuns: 500 })` for functions where rare edge cases matter.
3. Build custom strategies for domain objects rather than generating and filtering repeatedly.

**Python — custom strategy**
```python
from hypothesis import strategies as st

valid_emails = st.from_regex(r'[a-z]{1,10}@[a-z]{1,10}\.[a-z]{2,4}', fullmatch=True)

@given(valid_emails)
def test_email_parser_accepts_valid_emails(email):
    result = parse_email(email)
    assert result.domain is not None
```

**TypeScript — custom arbitrary**
```typescript
const validEmail = fc.stringOf(fc.char(), { minLength: 1, maxLength: 10 })
  .map(local => `${local}@example.com`);

test('parser accepts any valid email', () => {
  fc.assert(fc.property(validEmail, (email) => {
    expect(parseEmail(email).domain).toBe('example.com');
  }));
});
```

## Red Flags — Stop Immediately
- You are only running property tests with a `max_examples=1` or `numRuns=1` — that is an example test with extra syntax
- The property asserts on specific values rather than invariants (you have written an example test disguised as a property test)
- Every generated input is filtered out with `assume(False)` or `.filter(() => false)` — the strategy is too narrow
- You are mocking the function under test inside the property test

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "I already have example tests covering the happy path" | Example tests cover cases you thought of; property tests find the ones you missed |
| "The input space is too complex to generate" | Build a custom strategy for your domain type — Hypothesis and fast-check both support this |
| "Property tests are slow" | Hypothesis caches failures and re-runs them first; typical suites run in seconds |
| "I can't think of a property" | Start with round-trip or idempotency — almost every function has one |
| "The framework finds too many false positives" | Use `assume()` to enforce preconditions; do not remove the test |
| "We only need property tests for math-heavy code" | String parsing, serialisation, and validation all have large input spaces that benefit equally |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| Identify | Name the invariant in plain language | At least one property type chosen |
| Write | Implement with Hypothesis or fast-check | Test runs and framework generates inputs |
| Interpret | Read shrunk counterexample and fix production code | Framework reports no falsifying example |
| Tune | Adjust generators, constraints, and run count | Edge cases targeted; no excessive filtering |
