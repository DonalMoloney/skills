---
name: quick-review
description: Use when you need a fast, structured code review of the current working state — runs through a git diff workflow and checks for correctness, safety, and test coverage in under 5 minutes.
---

# Quick Review

## The Law
**Every change gets reviewed before it merges — a 5-minute review catches 80% of issues; skipping it because the change looks small is how subtle bugs ship.**

## When to Use
- Finishing work on a feature branch and preparing to open a PR
- After a focused bug fix before marking the ticket done
- A colleague asks for a fast second pair of eyes on a diff
- Before committing a change that touches shared utilities, auth, or data models
- **Never skip when:** the diff touches error handling, authentication, input validation, database writes, or any file with security implications — these categories warrant review regardless of diff size

## Process

### Phase 1: Scope the Diff
Goal — understand what changed before evaluating it.

```bash
# Staged changes only
git diff --cached

# All changes against the target branch
git diff main...HEAD

# Changed files at a glance
git diff --name-status main...HEAD

# Statistics — lines added/removed per file
git diff --stat main...HEAD
```

Python projects — also check for migration files:
```bash
git diff --name-only main...HEAD | grep -E "(migration|alembic|schema)"
```

1. Read the file list — categorise into: logic, tests, config, migrations, dependencies
2. Start with the file that carries the most risk, not the largest file
3. If the diff exceeds 400 lines of logic changes, flag it as too large for a quick review and recommend splitting

### Phase 2: Correctness Check
Goal — confirm the code does what it claims to do.

Work through each changed logic file:
1. Does the function or method do exactly one thing?
2. Are return types and error states handled explicitly — no silent swallowing?
3. Are edge cases covered: empty collections, null/None inputs, zero values, concurrent access?
4. Do variable and function names match what they actually do?
5. Are there any obvious off-by-one errors, reversed conditions, or copy-paste mistakes?

TypeScript — patterns that warrant a second look:
```typescript
// Red: swallowed error
try { await doThing(); } catch (_) {}

// Red: unchecked array access
const first = items[0].name; // what if items is empty?

// Green: explicit handling
if (items.length === 0) throw new Error("items is empty");
const first = items[0].name;
```

Python — equivalent patterns:
```python
# Red: bare except
try:
    do_thing()
except:
    pass

# Red: unchecked index
name = items[0]["name"]  # IndexError if empty

# Green: explicit handling
if not items:
    raise ValueError("items must not be empty")
name = items[0]["name"]
```

### Phase 3: Safety Check
Goal — confirm nothing dangerous was introduced.

5-minute safety checklist — answer each question:
1. Are any secrets, tokens, or credentials in the diff?
2. Are user inputs validated before they reach the database or filesystem?
3. Are SQL queries parameterised — no string interpolation into queries?
4. Does any new dependency have a known vulnerability? (`npm audit` / `pip-audit`)
5. Are error messages safe for external consumers — no stack traces or internal paths?

Quick secrets scan on changed files:
```bash
# List files changed in this branch then search for credential patterns
git diff --name-only main...HEAD | xargs grep -l -iE "secret|password|api_key|private_key" 2>/dev/null

# Dependency audit
npm audit --audit-level=high   # Node
pip-audit                       # Python
```

### Phase 4: Test Coverage Check
Goal — confirm the change has adequate test backing.

1. For every new function or class, find its test file
2. For every bug fix, find the test that would have caught the bug
3. Check that edge cases identified in Phase 2 have corresponding test cases
4. Run tests locally and confirm the suite is green:

```bash
# Node / TypeScript
npx jest --testPathPattern="<changed area>"

# Python
pytest tests/ -x -q --tb=short
```

5. If a test is missing for a meaningful logic path, note it — the review is not complete until coverage is addressed

### Phase 5: Summary
Goal — produce a concise verdict.

Write a review summary with three sections:
1. **Must fix** — items that block merge (incorrect logic, security issue, missing test for critical path)
2. **Should fix** — items that matter but are not blockers (naming, redundant code, missing edge case for non-critical path)
3. **Optional** — style or preference notes with no correctness implication

If there are zero must-fix items, the change is merge-ready.

## Red Flags — Stop Immediately
- A diff that touches both logic and tests but the test changes do not correspond to the logic changes
- A migration file with no corresponding rollback
- An added dependency that has not been audited
- Credentials or secrets visible anywhere in the diff, even in comments

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "It's a one-liner, no review needed" | One-line bugs in production are the most embarrassing kind |
| "I'll review it after merging" | Post-merge review finds bugs after they are in the main branch |
| "The tests passed so it's fine" | Tests only cover paths that were anticipated when they were written |
| "I wrote it, I know it's correct" | The author is the worst reviewer of their own work |
| "There's no time, we need to ship" | Skipping review trades minutes now for hours of debugging later |
| "The change is in a low-risk area" | Low-risk areas contain the bugs no one expects |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| Scope Diff | `git diff --stat`, categorise files | File list read, risk files identified |
| Correctness | Read logic files for error handling, edge cases | No silent failures or unchecked inputs |
| Safety | Secrets scan, input validation, dep audit | Zero credentials in diff, queries parameterised |
| Test Coverage | Match logic changes to test changes | Every new path has a test |
| Summary | Write must-fix / should-fix / optional | Zero must-fix items before approving |
