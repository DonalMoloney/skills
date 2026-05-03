---
name: map-test-coverage
description: Measure and map test coverage to find untested paths; use when coverage is unknown, coverage has dropped, or before a release to identify risk areas.
---

# Map Test Coverage

## The Law
**Coverage is a gap-finder, not a quality certificate — use it to locate untested paths, then decide which gaps matter, not to declare the code safe.**

## When to Use
- Coverage has never been measured on this codebase
- A PR drops coverage below the project threshold
- Preparing for a release and need to identify the riskiest untested paths
- After adding a new module to understand what the existing tests already exercise
- **Never skip when:** a "confidence" claim is being made about a module with no coverage data behind it

## Process

### Phase 1: Establish Tooling
1. Install the coverage tool for your language if not already present.
   - Python: `pip install pytest-cov`
   - TypeScript/Node: Istanbul is bundled with Jest as `--coverage`; for Vitest use `@vitest/coverage-v8`
2. Confirm the test suite passes before measuring — coverage over a broken suite is meaningless.
   - Python: `pytest -x`
   - TypeScript: `npx jest --ci`

### Phase 2: Generate the Coverage Report
1. Run the full suite with coverage enabled and output in a format you can inspect.
   - Python — terminal summary plus HTML:
     ```bash
     pytest --cov=src --cov-report=term-missing --cov-report=html tests/
     ```
     The `term-missing` flag prints which lines are uncovered directly in the terminal. The HTML report goes to `htmlcov/index.html`.
   - TypeScript — Jest:
     ```bash
     npx jest --coverage --coverageReporters=text-summary --coverageReporters=html
     ```
     HTML report lands in `coverage/lcov-report/index.html`.
   - TypeScript — Vitest:
     ```bash
     npx vitest run --coverage
     ```
2. Record the headline numbers: line coverage %, branch coverage %, function coverage %.
   - Branch coverage is more informative than line coverage — a line with an `if/else` can be "covered" even if the else branch never ran.

### Phase 3: Identify Coverage Gaps
1. Open the HTML report and sort by lowest coverage.
   - In `htmlcov` (pytest-cov) click a file to see red-highlighted uncovered lines
   - In Istanbul HTML click a file; red = uncovered line, yellow = partially covered branch
2. Triage each uncovered area into one of three buckets:
   - **Must cover** — business logic, error handling, security checks, data transformations
   - **Should cover** — utility helpers, configuration loaders, non-trivial branching
   - **Accept uncovered** — boilerplate entrypoints, generated code, intentional dead paths
3. Build a gap list: file path + line range + reason it matters.
   ```
   src/payments/processor.py  lines 88-102  — refund branch never exercised
   src/auth/token.py          line 45       — expiry check: expired-token path missing
   ```

### Phase 4: Set or Enforce a Threshold
1. Decide on a minimum acceptable coverage floor for the project.
   - A useful starting point for a codebase with no existing floor: measure current coverage, subtract 5 percentage points, set that as the threshold — it prevents regression without requiring perfection on day one.
2. Wire the threshold into the test runner so CI fails automatically on regressions.
   - Python (`pyproject.toml`):
     ```toml
     [tool.pytest.ini_options]
     addopts = "--cov=src --cov-fail-under=80 --cov-report=term-missing"
     ```
     Or pass inline: `pytest --cov=src --cov-fail-under=80`
   - TypeScript (`jest.config.js`):
     ```js
     module.exports = {
       collectCoverage: true,
       coverageThreshold: {
         global: {
           lines: 80,
           branches: 70,
           functions: 80,
         },
       },
     };
     ```
3. Commit the threshold configuration alongside the CI workflow change.

### Phase 5: Close Priority Gaps
1. Work through the "Must cover" gap list — write tests for each uncovered path.
   - Focus on behaviour, not lines: ask "what should the system do in this path?" not "how do I hit line 88?"
   - Python example — covering a previously untested refund path:
     ```python
     def test_refund_issued_when_order_cancelled_after_payment():
         order = Order(status="paid", total=Decimal("49.99"))
         result = processor.process_cancellation(order)
         assert result.refund_amount == Decimal("49.99")
         assert result.status == "refunded"
     ```
2. Re-run coverage after each batch of new tests to confirm the gaps close.
   ```bash
   pytest --cov=src --cov-report=term-missing tests/
   ```
3. Do not write tests that mechanically touch uncovered lines without asserting anything — that inflates the metric without improving confidence.

### Phase 6: Exclude Legitimately Dead Code
1. Mark genuinely untestable or irrelevant lines so they do not distort the report.
   - Python — inline pragma:
     ```python
     if TYPE_CHECKING:  # pragma: no cover
         from mypackage import SomeType
     ```
   - Python — `.coveragerc` exclude patterns:
     ```ini
     [report]
     exclude_lines =
         pragma: no cover
         if __name__ == .__main__.:
         raise NotImplementedError
     ```
   - TypeScript (`jest.config.js`):
     ```js
     coveragePathIgnorePatterns: ["/node_modules/", "/generated/", "index.ts"],
     ```
2. Keep exclusions minimal and reviewed — a sprawling exclusion list is a sign that real logic is being hidden from the metric.

## Red Flags — Stop Immediately
- Coverage is 95% but critical error-handling branches are all in the untested 5% — the number is hiding the risk
- Tests are written to hit specific lines rather than to assert specific behaviours — this is coverage gaming
- The `# pragma: no cover` comment is applied to entire functions or classes to make the number look better
- Branch coverage is never checked — line coverage alone passes a file with every `if` branch untested
- Threshold is set at 0% in CI, making it cosmetic only

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "We have 80% line coverage, we're fine" | Branch coverage can be 40% at the same time; check both |
| "The uncovered code is obvious, it doesn't need tests" | "Obvious" code is where assumptions live; assumptions are where bugs hide |
| "Adding a threshold will fail CI and block the team" | Set the initial threshold at current coverage minus 5%; it only blocks regressions, not existing debt |
| "Coverage reports take too long to generate" | `--cov-report=term-missing` adds under 2 seconds to most pytest runs; HTML is optional |
| "We'll exclude generated code later" | Add the exclude pattern now; generated code in the report distorts every decision made from it |
| "100% coverage means no bugs" | It means every line ran; it says nothing about whether the assertions were correct |
| "We only need to check coverage before a release" | Coverage checked only at release catches nothing until it is too late to fix it easily |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1. Tooling | Install pytest-cov or confirm Jest coverage | Tool installed, suite green |
| 2. Generate | Run with --cov-report=term-missing and HTML | Report on disk, headline numbers recorded |
| 3. Gap Triage | Sort by lowest coverage; classify each gap | Gap list with must/should/accept buckets |
| 4. Threshold | Set --cov-fail-under or coverageThreshold in config | CI fails on regression |
| 5. Close Gaps | Write behaviour tests for must-cover paths | Must-cover gaps closed, re-run green |
| 6. Exclude | pragma: no cover for genuinely dead code only | Exclusions minimal and reviewed |
