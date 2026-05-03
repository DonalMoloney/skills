---
name: fix-lint
description: Run auto-fix commands for Python and JavaScript/TypeScript linting and formatting tools in sequence, then report what remains.
---

# Fix Lint

## The Law
**Run every applicable auto-fix tool in the correct order before touching a single issue by hand — automated fixes are faster, consistent, and leave a clean diff.**

## When to Use
- After `auto-lint` reports fixable issues
- Before opening a pull request on a project with linting rules
- After merging a branch that introduced style drift
- When bringing a legacy file up to current standards
- **Never skip when:** the CI pipeline enforces formatting (Black, Prettier, Ruff format) — a failing format check blocks everyone

## Process

### Phase 1: Confirm Scope
1. Determine which language(s) are present (see `auto-lint` if unsure)
2. Check that the project has a lock file or dependency manifest before running tools
   - Python: `pyproject.toml`, `setup.cfg`, or `requirements.txt`
   - JS/TS: `package.json`
3. Confirm which tools are installed:
   ```bash
   # Python
   which ruff && ruff --version
   which black && black --version
   which isort && isort --version

   # JS/TS
   npx eslint --version
   npx prettier --version
   ```
4. If a tool is missing, install it before proceeding — do not skip it silently

### Phase 2: Run Python Fixes (if Python project)
Run in this exact order — sequence matters because each tool affects what the next one sees:

1. **isort** — sorts and groups imports first so Black does not undo import changes:
   ```bash
   isort .
   ```

2. **Black** — formats all Python files to a consistent style:
   ```bash
   black .
   ```

3. **Ruff (fix mode)** — catches remaining lint issues Black does not handle:
   ```bash
   ruff check . --fix
   ```

4. **Ruff (format mode)** — if the project uses Ruff as the formatter instead of Black:
   ```bash
   ruff format .
   ```
   Note: do not run both `black` and `ruff format` on the same project — pick one.

5. Run the linter one more time to see what remains:
   ```bash
   ruff check .
   ```

### Phase 3: Run JavaScript / TypeScript Fixes (if JS/TS project)
Run in this exact order:

1. **ESLint with --fix** — handles rule violations that have auto-fixers:
   ```bash
   npx eslint . --fix --ext .ts,.tsx,.js,.jsx
   ```

2. **Prettier** — reformats files for consistent style after ESLint changes:
   ```bash
   npx prettier --write .
   ```

3. Run ESLint once more (without --fix) to see what requires manual attention:
   ```bash
   npx eslint . --ext .ts,.tsx,.js,.jsx
   ```

### Phase 4: Handle Manual-Fix Issues
1. List every issue the auto-fix tools could not resolve
2. Group them by file and rule name
3. Prioritise errors over warnings — fix errors before committing
4. For each remaining error:
   - Read the rule documentation before changing code
   - Make the minimal change that satisfies the rule
   - Re-run the linter on that file to confirm the fix
5. Do not suppress issues with `# noqa`, `// eslint-disable`, or `type: ignore` unless the rule is genuinely inapplicable and document why inline

### Phase 5: Verify the Result
1. Run the full linter suite one final time — output should be clean or show only known accepted warnings:
   ```bash
   # Python
   ruff check . && black --check .

   # JS/TS
   npx eslint . && npx prettier --check .
   ```
2. Run the test suite to confirm no behaviour changed:
   ```bash
   # Python
   pytest -q

   # JS/TS
   npm test
   ```
3. Report: issues fixed automatically, issues fixed manually, issues remaining with justification

## Red Flags — Stop Immediately
- Black and isort produce conflicting changes that loop — check for conflicting config values (`profile = "black"` must be set in isort config)
- ESLint --fix is removing code rather than reformatting it — a rule is misconfigured
- Prettier is overwriting intentional formatting in generated files — add those paths to `.prettierignore`
- A test breaks after an auto-fix — the fix changed semantics, revert that file and fix manually
- More than 50 manual-fix issues remain after running all tools — this file needs a dedicated cleanup session, not a pre-commit patch

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "I'll just run Black and call it done" | Black handles formatting only; Ruff and isort catch separate classes of issues |
| "Order doesn't matter for these tools" | isort after Black can re-break import formatting; sequence is deliberate |
| "I'll add a noqa and move on" | Suppressions accumulate and obscure real problems; fix the issue or fix the config |
| "The formatter changes too much — harder to review" | Run the formatter as a standalone commit before the feature commit |
| "This file is auto-generated, leave it" | Add it to `.prettierignore` or `exclude` in ruff config rather than skipping globally |
| "ESLint --fix changed my logic" | ESLint --fix is conservative; if it changed logic a rule is misconfigured — investigate before committing |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1. Scope | Confirm tools installed and language identified | All tools available, none missing |
| 2. Python | isort → Black → Ruff --fix → Ruff check | No auto-fixable issues remain |
| 3. JS/TS | ESLint --fix → Prettier → ESLint check | No auto-fixable issues remain |
| 4. Manual | Fix remaining errors, document suppressions | Only accepted warnings left |
| 5. Verify | Full linter and test suite pass | Green linter, green tests |
