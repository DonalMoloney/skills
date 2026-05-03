---
name: tdd-handbook
description: Team reference for the TDD cycle — use to onboard engineers, resolve disputes about test-first discipline, or re-establish the practice after it has drifted.
---

# TDD Playbook

## The Law
**No production code exists without a failing test that demanded it — the test must come first, every time, on every feature, from every engineer on the team.**

## When to Use
- Onboarding a new engineer to the team's testing standards
- The team has drifted back to write-code-then-test habits and needs a reset
- Resolving a disagreement about whether a particular change "needs" a test first
- Starting a new module or service where the team gets to set the standard from scratch
- **Never skip when:** there is time pressure — pressure is exactly when discipline pays off most, and when it is most likely to be abandoned

## Process

### Phase 1: Write the Failing Test
1. Before touching any production code, write a single test that expresses one small, specific behaviour.
   - The test must be for behaviour, not for implementation: "given X input, the function produces Y output" not "the function calls this helper"
   - Python (pytest):
     ```python
     def test_invoice_calculates_vat_at_standard_rate():
         invoice = Invoice(subtotal=Decimal("100.00"), vat_rate=Decimal("0.20"))
         assert invoice.total == Decimal("120.00")
     ```
   - TypeScript (Jest):
     ```typescript
     it("calculates VAT at the standard rate", () => {
       const invoice = new Invoice({ subtotal: 100.00, vatRate: 0.20 });
       expect(invoice.total).toBe(120.00);
     });
     ```
2. Run the test and confirm it fails — not with an import error or syntax error, but with an assertion failure or a clear "not implemented" signal.
   - A failing import means the interface does not exist yet; create the stub so the test fails on the assertion, then stop.
3. Commit the failing test if the team uses a "red commit" convention — it locks in the requirement as code.

### Phase 2: Write the Minimum Code to Pass
1. Write only enough production code to make the failing test pass.
   - Resist the urge to write a complete solution — write the smallest correct implementation.
   - If the temptation is to write more, ask: "Do I have a test demanding this behaviour?" If not, stop.
2. Run the test again — it must pass before moving to Phase 3.
3. Run the full suite — no previously passing test should now be failing.

### Phase 3: Refactor
1. With all tests green, clean the code: remove duplication, clarify names, extract helpers.
   - Refactoring rule: the test suite must remain green throughout every refactoring step.
   - Python: run `pytest` after each small change, not just at the end.
   - TypeScript: run `npx jest --watchAll` and watch for the first red flash.
2. Do not add new behaviour during refactor — that belongs in Phase 1 of the next cycle.
3. When refactoring is complete, run the full suite once more and confirm everything is green.

### Phase 4: Repeat
1. Pick the next smallest behaviour that needs to exist and return to Phase 1.
2. Each cycle should take minutes, not hours — if a cycle is taking an hour, the test is too large; split it.

## Team Agreements
These are the non-negotiable decisions the team must make once and record in `CLAUDE.md` or the project wiki:

1. **Test location** — co-located (`src/foo.test.ts`) or separate directory (`tests/test_foo.py`)? Pick one, enforce it consistently.
2. **Naming convention** — `test_<behaviour>` (Python) or `it("<does something>")` (TypeScript); names describe the expected behaviour, not the implementation.
3. **Red commit convention** — does the team commit failing tests? Decide. If yes, use a branch; failing commits never land on main.
4. **Minimum cycle size** — one behaviour per cycle; if a cycle covers more than one observable behaviour, it needs to be split.
5. **Mocking boundary** — the team decides what gets mocked (external I/O, third-party APIs) and what must not (domain logic); document this boundary.
6. **Coverage floor** — agree on a branch coverage minimum and enforce it in CI; the default recommendation is 80%.

## Common Failure Modes
These are the ways TDD practice breaks down in teams. Recognizing them early stops the drift.

1. **Test after** — engineers write code, then write tests to cover it. The result is tests that never failed, which means they prove nothing.
2. **Test the implementation** — tests that assert `mock_helper.assert_called_once()` instead of asserting the observable output. These tests break on refactoring and add no confidence.
3. **Skipping refactor** — the cycle stops at green without cleaning. Technical debt accumulates cycle by cycle.
4. **Omnibus tests** — one test method exercises three or four behaviours. When it fails, it is unclear which behaviour broke.
5. **Mocking everything** — heavy mocking means the test suite passes but the integration path is never exercised. Reserve mocks for genuine external dependencies.
6. **Slow test suite** — when the full suite takes more than 60 seconds, engineers stop running it between cycles. Keep the suite fast or use `pytest -x` to run only affected tests locally.

## Red Flags — Stop Immediately
- A PR adds production code with no new or modified tests — send it back without review
- A test was written after the production code was complete — flag it; at minimum, delete the implementation, run the test to confirm it fails, then re-introduce the code
- Tests are being deleted to make CI green — this is the most serious violation; deleted tests hide real failures
- The team is debating whether a given change "is big enough to need a test first" — the answer is always yes
- A cycle has been running for more than two hours — the scope is too large; stop and split

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "We're moving fast and TDD slows us down" | TDD's cost is seconds per cycle; the cost of debugging code without tests is hours — the math favors discipline |
| "This code is too simple to test first" | Simple code has the lowest test-writing cost; there is no simpler situation in which to build the habit |
| "We'll write tests at the end when the design is stable" | Writing tests after means the design was never challenged by a consumer perspective; tests written first reveal design flaws early |
| "The test suite is already slow" | That is a reason to fix the suite, not to skip writing tests; a slow suite is a separate problem |
| "Mocking this module is too complex" | Complexity in mocking is a signal that the module has too many dependencies; TDD is surfacing a design problem |
| "We do code review, that catches problems" | Code review catches style and obvious bugs; it does not verify behaviour under edge cases the way tests do |
| "Our product changes too fast for stable tests" | Tests written against stable behaviour survive churn; tests written against implementation break with every refactor |
| "I already know what the code should do" | Writing the test forces you to express that knowledge as a precise, executable specification — that process reliably reveals assumptions |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1. Red | Write test for one behaviour; run it; confirm it fails | Test fails with assertion error, not import/syntax error |
| 2. Green | Write minimum production code to pass the test | Test passes; full suite still green |
| 3. Refactor | Clean code while keeping suite green | No duplication; names clear; suite still green |
| 4. Repeat | Return to Phase 1 with next behaviour | All agreed behaviours have tests; suite green |
