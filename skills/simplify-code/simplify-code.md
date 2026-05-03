---
name: simplify-code
description: After writing or modifying a logical chunk of code, review it for unnecessary complexity, duplication, and naming issues, then fix what you find.
---

# Simplify Code

## The Law
**Every code change must be followed by a clarity pass — readable, explicit code that any teammate can maintain beats compact code that only the author understands.**

## When to Use
- After implementing a new feature or function
- After fixing a bug with conditional logic
- After a refactor that touched multiple files
- After any session where "it works" came before "it's clean"
- **Never skip when:** the change introduced new nesting levels, conditional chains, or copied logic from elsewhere

## Process

### Phase 1: Scope the Work
1. Identify which files and functions were modified in the current session
   - Use `git diff --name-only` or review recent edits
   - Limit scope to touched code unless explicitly asked for a full pass
2. Read each modified section end-to-end before making any changes
3. Note: naming problems, dead branches, duplicated logic, deep nesting, and stale comments

### Phase 2: Rename for Clarity
1. Replace single-letter or abbreviated names with descriptive ones
   - Python: `d` → `user_data`, `fn` → `process_payment`
   - TypeScript: `cb` → `onSuccess`, `res` → `apiResponse`
2. Align names with the domain language used elsewhere in the codebase
3. Rename booleans to read as assertions: `loaded` → `isLoaded`, `err` → `hasError`

### Phase 3: Extract and Consolidate
1. Pull repeated logic into a named function
   - Python example — before:
     ```python
     # in two separate places
     items = [x for x in raw if x.get("active") and x["score"] > 0]
     ```
   - Python example — after:
     ```python
     def eligible_items(raw: list[dict]) -> list[dict]:
         return [x for x in raw if x.get("active") and x["score"] > 0]
     ```
   - TypeScript example — before:
     ```typescript
     const items = raw.filter(x => x.active && x.score > 0)
     ```
   - TypeScript example — after:
     ```typescript
     function eligibleItems(raw: Item[]): Item[] {
         return raw.filter(item => item.active && item.score > 0);
     }
     ```
2. Collapse nested conditionals where the intent remains obvious
   - Prefer early returns over deeply indented success paths
3. Remove abstractions that add indirection without adding clarity

### Phase 4: Remove Dead Code
1. Delete unreachable branches confirmed by tracing control flow
2. Remove commented-out code that predates the current session
3. Strip comments that restate what the code already says clearly
   - Keep comments that explain *why*, remove ones that explain *what*

### Phase 5: Verify Nothing Changed
1. Run the test suite before and after — output must be identical
   - Python: `pytest` or `python -m pytest -q`
   - TypeScript/Node: `npm test` or `npx jest`
2. Run the linter to confirm no new warnings were introduced
   - Python: `ruff check .` or `flake8`
   - TypeScript: `npx eslint .`
3. If any test fails, revert that specific simplification — do not push through

## Red Flags — Stop Immediately
- A test that was passing is now failing after a rename or extraction
- You are about to consolidate two functions that handle subtly different edge cases
- The "simplified" version is shorter but requires reading it twice to understand
- You are removing a comment on a non-obvious algorithm to make the file look cleaner
- Scope creep: you are editing files that were not part of the original change

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "The nested ternary saves three lines" | Nested ternaries fail code review every time; use if/else or a match statement |
| "That dead code might be useful later" | Version control exists for that reason; dead code costs readers every day it stays |
| "Renaming breaks git blame" | Clarity for the next six months outweighs one noisy blame line |
| "I'll clean it up in a follow-up PR" | Follow-up PRs for cleanup rarely land; do it now while context is fresh |
| "It's obvious what this does" | Obvious to you today, opaque to a reviewer next week |
| "Extracting a function adds a call stack frame" | The overhead is immeasurable; the readability gain is immediate |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1. Scope | Identify touched files and sections | List of targets confirmed |
| 2. Rename | Names match domain language and read clearly | No abbreviations or single-letter names remain |
| 3. Extract | Repeated logic lives in one named function | No duplicated blocks |
| 4. Dead Code | Unreachable and stale code removed | No commented-out blocks or unused branches |
| 5. Verify | Tests pass, linter clean | Green suite, zero new warnings |
