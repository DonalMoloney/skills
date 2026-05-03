---
name: coverage-gaps
description: Use to review test coverage in a PR — identify untested code paths, missing edge cases, and silent failure points. Output a coverage report with gaps and risk assessment.
---

# Coverage Gaps

## The Law
**100% line coverage is not 100% path coverage. Tests that touch a line without exercising its failure modes are not real tests. Aim for path coverage, not line coverage — every decision point tested both ways.**

## When to Use
- A PR adds new functionality and needs test coverage review
- Coverage reports show high percentage but gaps remain
- You need to verify that error handling is actually tested
- **Never skip when:** the code touches production logic or user data

## Process

### Phase 1: Identify Coverage Holes
1. Run coverage tool for the changed files:
   - **Python:** `pytest --cov=path/to/file --cov-report=html`
   - **JavaScript:** `npm test -- --coverage path/to/file`
2. Look for red (uncovered) lines
3. Distinguish between:
   - **Unreachable code** (dead code)
   - **Untested happy path** (should have tests)
   - **Untested failure paths** (should have tests)

### Phase 2: Classify Gaps by Risk
1. **HIGH RISK** — Error handling, validation, null checks, security boundaries
2. **MEDIUM RISK** — Alternate paths, unusual inputs, logging
3. **LOW RISK** — Comments, formatting, type hints

### Phase 3: Write Missing Tests
1. For each HIGH RISK gap, write a test
2. For each MEDIUM RISK gap, write a test unless reason is documented
3. For LOW RISK gaps, document why they're acceptable (comments in test file)

### Phase 4: Generate Coverage Report
1. Run coverage again on all changed files:
   ```bash
   pytest --cov=path coverage-report.txt
   ```
2. Output:
   ```
   COVERAGE REPORT
   File: path/to/file.py
   Coverage: X%
   
   Gaps (HIGH): <list>
   Gaps (MEDIUM): <list>
   
   Verdict: PASS | FAIL
   ```

## Red Flags — Stop Immediately

- Coverage below 80% for production code — flag it
- Error handling is untested — flag it as HIGH RISK
- Validation code has gaps — flag it as HIGH RISK
- Test assertions don't verify the failure case — the test is incomplete
- "This path never happens" without a guard clause — add the guard or test it

## Common Rationalizations

| Excuse | Why It's Wrong |
|--------|---|
| "Line coverage is high enough" | Line coverage doesn't test both sides of conditionals. Path coverage does. |
| "The error case never happens in practice" | If it never happens, why is the code there? If it can happen, test it. |
| "We don't have time to test error paths" | Untested error paths fail in production. Testing takes less total time. |
| "This test is flaky, let's skip it" | Flaky tests reveal real bugs. Fix the flakiness, don't skip the test. |
| "The coverage tool says we're good" | The tool measures lines, not paths. You measure quality. |

## Quick Reference

| Phase | Core Action | Done When |
|-------|---|---|
| **Identify** | Run coverage tool; mark uncovered lines | All uncovered lines are visible |
| **Classify** | Sort gaps by risk: HIGH / MEDIUM / LOW | Gaps are ranked by impact |
| **Test** | Write tests for HIGH and MEDIUM gaps | All HIGH gaps have tests |
| **Report** | Generate coverage report; output verdict | PASS or FAIL is explicit |
