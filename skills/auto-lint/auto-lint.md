---
name: auto-lint
description: Use to auto-fix linting issues — detect language, run language-specific fixer (Black, Prettier, eslint --fix), verify tests still pass. One command to clean up formatting.
---

# Auto Lint

## The Law
**Linting and formatting are separate concerns. Linters detect issues; formatters fix formatting automatically. Use formatters for style, linters for logic errors.**

## When to Use
- Code has formatting issues (spacing, quotes, line length)
- You want consistent style across the codebase
- Before committing — clean up formatting first
- **Never skip when:** joining a team project — style consistency matters

## Process

### Phase 1: Detect Language
1. Check file extensions in the modified files:
   - `.py` → Python
   - `.js`, `.ts` → JavaScript/TypeScript
   - `.go` → Go
   - `.rb` → Ruby
2. Check if a config file exists:
   - `pyproject.toml`, `.black.toml` (Python)
   - `.prettierrc`, `prettier.config.js` (JS/TS)
   - `.eslintrc.json` (JS)
   - `.golangci.yml` (Go)

### Phase 2: Run Formatter
1. **Python:**
   ```bash
   black .
   isort .
   ```
2. **JavaScript/TypeScript:**
   ```bash
   npx prettier --write .
   ```
3. **Go:**
   ```bash
   go fmt ./...
   gofmt -s -w .
   ```
4. **Ruby:**
   ```bash
   bundle exec rubocop -a
   ```

### Phase 3: Run Linter
1. **Python:**
   ```bash
   ruff check . --fix
   ```
2. **JavaScript/TypeScript:**
   ```bash
   npx eslint . --fix
   ```
3. **Go:**
   ```bash
   golangci-lint run ./... --fix
   ```

### Phase 4: Verify Tests Pass
1. Run the test suite:
   ```bash
   npm test  # or pytest, go test, etc.
   ```
2. If tests fail, the formatting broke something — investigate
3. If all pass, commit the formatted code

### Phase 5: Commit Clean Code
1. Stage the formatted files:
   ```bash
   git add .
   ```
2. Commit with a clear message:
   ```bash
   git commit -m "style: auto-format code with prettier and eslint"
   ```

## Red Flags — Stop Immediately

- Tests fail after formatting — the formatter may have introduced logic errors
- Formatter deleted important code — something is wrong with your config
- Formatting changes are enormous — check your config, you might have the wrong rules
- You don't have a formatter config — create one before running auto-fix

## Common Rationalizations

| Excuse | Why It's Wrong |
|--------|---|
| "Manual formatting is faster" | It's not, and it's inconsistent. Automation is always faster. |
| "The formatting is fine, we don't need a formatter" | Inconsistent formatting distracts reviewers. Automation solves this. |
| "Formatters sometimes mess up code" | They shouldn't if your config is correct. Fix the config. |
| "We'll format before merge" | Do it continuously. Don't let formatting debt accumulate. |

## Quick Reference

| Phase | Core Action | Done When |
|-------|---|---|
| **Detect** | Identify language from files and config | Language is determined |
| **Format** | Run language-specific formatter (Black, Prettier, etc.) | Code is reformatted consistently |
| **Lint** | Run language-specific linter with --fix | Logic issues are fixed |
| **Test** | Run test suite; verify all pass | Tests pass after formatting |
| **Commit** | Stage and commit formatted code | Code is clean and committed |
