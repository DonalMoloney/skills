---
name: mutation-test
description: Run mutation testing to verify that the test suite actually catches code defects; use when coverage looks healthy but confidence is low, or before a release gate.
---

# Mutation Test

## The Law
**A green test suite that cannot catch deliberate code defects is a liability, not an asset — mutation score must reach 80% or the suite is not trusted.**

## When to Use
- Coverage metrics look fine but bugs keep escaping into production
- Before tagging a release on a critical module
- After a test suite refactor to confirm nothing was hollowed out
- Any module handling money, auth, or data integrity
- **Never skip when:** the team is tempted to treat line coverage as a proxy for test quality — it is not

## Process

### Phase 1: Baseline Run
1. Confirm the existing test suite passes cleanly with no skipped tests.
   - Python: `pytest -x` — fix any failures before proceeding
   - TypeScript: `npx jest --ci` — fix any failures before proceeding
2. Record the current line/branch coverage figure as a baseline for comparison.
3. Install the mutation tool if not already present.
   - Python: `pip install mutmut`
   - TypeScript: `npm install --save-dev @stryker-mutator/core @stryker-mutator/jest-runner`

### Phase 2: Configure the Runner
1. Scope the run to the module under test — running against the entire codebase on the first pass takes too long.
   - Python (`mutmut`): add a `[mutmut]` section to `setup.cfg` or `pyproject.toml`:
     ```toml
     [mutmut]
     paths_to_mutate = src/billing/
     tests_dir = tests/
     runner = python -m pytest -x --timeout=10
     ```
   - TypeScript (`stryker.config.json`):
     ```json
     {
       "testRunner": "jest",
       "mutate": ["src/billing/**/*.ts", "!src/billing/**/*.test.ts"],
       "coverageAnalysis": "perTest",
       "thresholds": { "high": 80, "low": 60, "break": 60 }
     }
     ```
2. Set a per-test timeout — mutmut and Stryker both need it to avoid hanging on infinite loops introduced by mutations.

### Phase 3: Run and Read the Report
1. Execute the mutation run.
   - Python: `mutmut run`
   - TypeScript: `npx stryker run`
2. Check the mutation score once complete.
   - Python: `mutmut results` — shows survived, killed, and timeout counts
   - TypeScript: open `reports/mutation/mutation.html` in a browser
3. Calculate score: `killed / (killed + survived) * 100`. Target is 80% or above.
4. List the survived mutants — these are the ones that matter:
   - Python: `mutmut show <id>` for each survived mutant
   - TypeScript: the HTML report flags survivors by file and line

### Phase 4: Close the Gaps
1. For each survived mutant, determine why no test caught it.
   - Common causes: assertion too loose (`assert result is not None` instead of `assert result == expected_value`), happy-path-only tests, no edge-case test for the mutated operator
2. Write the tightest possible test that kills the mutant — test behaviour, not implementation.
   - Python example: a mutant that changed `>` to `>=` in a discount threshold survived because tests only tested values well inside the boundary; add a test at the exact boundary value
     ```python
     def test_discount_not_applied_at_threshold_boundary():
         # mutant changed > to >= — this test kills it
         cart = Cart(total=Decimal("100.00"))
         assert apply_discount(cart) == Decimal("0.00")

     def test_discount_applied_just_above_threshold():
         cart = Cart(total=Decimal("100.01"))
         assert apply_discount(cart) == Decimal("5.00")
     ```
   - TypeScript example:
     ```typescript
     it("does not apply discount at the exact threshold", () => {
       expect(applyDiscount({ total: 100.00 })).toBe(0);
     });
     it("applies discount one cent above threshold", () => {
       expect(applyDiscount({ total: 100.01 })).toBe(5);
     });
     ```
3. Re-run mutation testing after adding new tests to confirm the mutants are now killed.

### Phase 5: Gate and Record
1. If mutation score is below 80%, block the release — do not lower the bar.
2. Commit both the new tests and the tool configuration.
3. Add the mutation score to the team's quality dashboard or CI output for trend tracking.
4. Schedule a re-run whenever the module changes significantly.

## Red Flags — Stop Immediately
- Score is above 80% but you achieved it by widening assertions to accept any output — that defeats the point entirely
- Survived mutants are in authentication, authorization, or payment logic — those must be killed regardless of overall score
- Timeouts account for more than 10% of the run — your test suite has sleeps or external calls that need to be mocked
- The team is only running mutation tests on new code and treating legacy code as untouchable
- A mutant survives because the test was deleted rather than fixed

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "We already have 90% line coverage" | Line coverage proves code ran; mutation testing proves code was checked — they measure different things |
| "Mutation testing is too slow" | Scope it to the riskiest modules; a targeted 10-minute run beats trusting hollow coverage forever |
| "The survived mutants are in trivial code" | Trivial code has introduced critical bugs; no module is exempt from the 80% gate |
| "We'll add it after the release" | Confidence debt compounds — run it before the release, not after |
| "80% is arbitrary" | It is a floor, not a ceiling; start there and raise it as the team builds the habit |
| "Adding tests just to kill mutants feels artificial" | Those tests catch the exact class of boundary bug that slips through line-coverage-only suites |
| "Our framework makes mutation testing hard" | That is a tooling problem to solve, not a reason to skip the practice |
| "Stryker/mutmut configuration is too complex" | The setup-mutation-tests skill walks through it step by step |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1. Baseline | Clean test run; install tool | Suite green, tool installed |
| 2. Configure | Scope to module; set timeout | Config file committed |
| 3. Run and Read | Execute run; list survivors | Mutation score calculated |
| 4. Close Gaps | Write tests to kill survivors | Re-run shows all targeted mutants killed |
| 5. Gate | Block release below 80%; record score | Score >= 80%, committed, tracked |
