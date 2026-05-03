---
name: mutation-setup
description: Wire mutation testing into an existing repository that has never run it before; use when adding mutmut (Python) or Stryker (TypeScript) to a project for the first time.
---

# Setup Mutation Tests

## The Law
**Mutation testing must be a first-class citizen in CI from day one of setup — a tool that only runs locally will drift and eventually stop running.**

## When to Use
- A repo has no mutation testing configuration at all
- Onboarding mutation testing as a quality gate for a new module or service
- Migrating from an older mutation tool to mutmut or Stryker
- **Never skip when:** the team has agreed to adopt mutation testing but "someone will set it up later" — later never arrives

## Process

### Phase 1: Audit What Already Exists
1. Confirm the test suite is passing cleanly — do not add mutation tooling on top of broken tests.
   - Python: `pytest -x` must exit 0
   - TypeScript: `npx jest --ci` must exit 0
2. Check if any mutation tooling is already partially configured.
   - Python: look for `mutmut` in `requirements*.txt`, `pyproject.toml`, or `setup.cfg`
   - TypeScript: look for `@stryker-mutator` in `package.json` dependencies
3. Identify the highest-risk module to target first — the one where a bug would hurt most (auth, billing, core domain logic).

### Phase 2: Install the Tool
**Python — mutmut:**
1. Add `mutmut` to your dev dependencies.
   ```bash
   pip install mutmut
   # or if using pip-tools / pyproject.toml:
   # add mutmut to [project.optional-dependencies] dev
   ```
2. Verify the install: `mutmut --version`

**TypeScript — Stryker:**
1. Use the Stryker init wizard to bootstrap configuration.
   ```bash
   npm install --save-dev @stryker-mutator/core
   npx stryker init
   ```
   - The wizard asks for your test runner (Jest, Vitest, Mocha) and outputs a `stryker.config.json`.
2. Install the appropriate test runner plugin, e.g.:
   ```bash
   npm install --save-dev @stryker-mutator/jest-runner
   ```

### Phase 3: Write the Configuration File
**Python — `pyproject.toml` (preferred) or `setup.cfg`:**
```toml
[mutmut]
paths_to_mutate = src/payments/
tests_dir = tests/
runner = python -m pytest -x --timeout=10 -q
dict_synonyms = Equiv, Equivalent
```
Key settings to get right:
- `paths_to_mutate` — point only at the module you chose in Phase 1, not the entire `src/`
- `--timeout` — prevents mutants that introduce infinite loops from hanging the entire run
- `-q` — quiet output so mutmut can parse results reliably

**TypeScript — `stryker.config.json`:**
```json
{
  "testRunner": "jest",
  "jest": { "configFile": "jest.config.js" },
  "mutate": [
    "src/payments/**/*.ts",
    "!src/payments/**/*.spec.ts",
    "!src/payments/**/*.test.ts"
  ],
  "coverageAnalysis": "perTest",
  "timeoutMS": 10000,
  "thresholds": {
    "high": 80,
    "low": 60,
    "break": 60
  },
  "reporters": ["html", "clear-text", "progress"]
}
```
Key settings to get right:
- `coverageAnalysis: "perTest"` — dramatically speeds up the run by only running the tests relevant to each mutant
- `thresholds.break: 60` — Stryker will exit non-zero if score drops below this; use this in CI
- Exclude test files from `mutate` — Stryker does not mutate them by default but being explicit avoids surprises

### Phase 4: Run a Dry-Run to Verify Configuration
1. Run against a single file first to confirm the setup works end-to-end before a full run.
   - Python:
     ```bash
     # mutmut does not have a single-file mode, so scope via paths_to_mutate temporarily
     mutmut run --paths-to-mutate src/payments/calculator.py
     mutmut results
     ```
   - TypeScript:
     ```bash
     npx stryker run --mutate src/payments/calculator.ts
     ```
2. Confirm:
   - At least one mutant was generated
   - At least one mutant was killed (proves the test suite is connected)
   - No timeouts on the first run (if there are many, shorten `--timeout` or mock external calls)
3. Fix any configuration errors before proceeding to a full module run.

### Phase 5: Add to CI
**GitHub Actions — Python:**
```yaml
- name: Run mutation tests
  run: |
    pip install mutmut
    mutmut run
    mutmut results
  continue-on-error: false   # fail the build if score drops
```
For a score gate, capture the results and fail explicitly:
```yaml
- name: Mutation score gate
  run: |
    mutmut run
    python -c "
    import subprocess, sys
    out = subprocess.check_output(['mutmut', 'results']).decode()
    lines = [l for l in out.splitlines() if 'survived' in l or 'killed' in l]
    # parse and assert score >= 80
    "
```
Alternatively, integrate with `mutmut-exporter` or `pytest-mutmut` for structured output.

**GitHub Actions — TypeScript:**
```yaml
- name: Run Stryker mutation tests
  run: npx stryker run
  # stryker exits non-zero when score < thresholds.break
```

### Phase 6: Document and Hand Off
1. Add a short note to the project `CLAUDE.md` or `CONTRIBUTING.md` explaining how to run mutation tests locally and what the score gate is.
2. Commit the configuration file (`pyproject.toml` change or `stryker.config.json`) and the CI workflow addition together in one atomic commit.
3. Record the baseline mutation score so the team can track trends — even a note in the commit message is enough to start.

## Red Flags — Stop Immediately
- The test suite is failing before mutation testing is added — fix tests first, always
- The first dry-run produces zero mutants — the `paths_to_mutate` or `mutate` pattern is wrong; check the path
- Every mutant times out — the tests call real external services; mock them before running mutation tests
- CI is set to `continue-on-error: true` on the mutation step — this turns the gate into a suggestion
- The tool is configured but the score threshold is 0% — a threshold of 0% is the same as no threshold

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "Setup takes too long, we'll do it next sprint" | The wizard plus a single dry-run takes under 30 minutes; the "next sprint" debt compounds indefinitely |
| "Our tests are fast and we can't afford the mutation run time" | `coverageAnalysis: perTest` (Stryker) and module-scoped runs (mutmut) cut run time by 60-80% |
| "The config is too complex to get right" | This skill provides a working config for both tools; copy, adjust the path, run |
| "We only need it for new code" | Legacy code is where the most dangerous bugs hide; at minimum run it on any code touched in a PR |
| "CI is already slow" | Add mutation testing as a non-blocking parallel job first, then make it blocking once the score is stable |
| "We don't have the score data to set a meaningful threshold" | Run it once, record the score, set the threshold 10 points below that — you now have a floor |
| "Stryker/mutmut breaks on our framework" | Open a scoped issue; the tools support most mainstream frameworks — check docs before giving up |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1. Audit | Confirm green suite; pick target module | Tests passing, module identified |
| 2. Install | pip install mutmut / npx stryker init | Tool installed and verified |
| 3. Configure | Write config with scope and timeout | Config file on disk |
| 4. Dry-Run | Run against one file; confirm mutants generated and killed | At least one kill, no timeouts |
| 5. CI | Add mutation step with score gate | CI fails below threshold |
| 6. Document | Commit config + note baseline score | Config committed, score recorded |
