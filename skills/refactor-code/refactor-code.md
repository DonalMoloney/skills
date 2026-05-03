---
name: refactor-code
description: Use when code needs clarity — clean names, single responsibility, no duplication. Before changing anything, state the plan and wait for confirmation on medium-risk changes.
---

# Refactor Code

## The Law
**Code clarity is non-negotiable. Every variable, function, and class must say what it does without requiring a comment to decode. If a name requires context, rename it.**

## When to Use
- A function or class has grown unclear and requires explanation to new readers
- You notice repeated logic patterns that could be extracted
- Names hide intent (e.g., `valid` instead of `isEmailVerified`, `process` instead of `convertCsvToJson`)
- Complex conditionals nest too deeply or mix concerns
- **Never skip when:** preparing code for a code review — unreadable code kills productivity downstream

## Process

### Phase 1: State the Refactoring Plan

Before any changes, output:

```
REFACTORING PLAN
Target: <file or function name>
Smells found: <list each issue — unclear names, duplication, complexity, dead code>
Planned changes: <what will change and why>
Risk: LOW | MEDIUM
```

- **LOW risk:** rename variables, extract small functions, remove dead code
- **MEDIUM risk:** split a function into multiple functions, reorganize a class
- Wait for confirmation if risk is MEDIUM or higher. Proceed immediately if risk is LOW.

### Phase 2: Identify Code Smells

Work through the code in this order. Stop and note each finding before changing it.

**1. Name Clarity**
- Variables, functions, and classes should say what they hold or do without a comment
- Rename anything where the name requires context to decode
- Prefer `getUserById` over `getUser`, `isEmailVerified` over `valid`

**2. Single Responsibility**
- A function that does more than one thing should be split
- A class with unrelated methods should be decomposed
- Each unit should be nameable with a single verb phrase

**3. Duplication**
- Extract repeated logic into a named function — never copy-and-fix
- Suggest a name for every extracted function before writing it
- Do not inline: if the extracted unit has a name, keep it separate

**4. Dead Code**
- Remove unreachable branches, unused imports, commented-out blocks
- Do not keep stubs or `// TODO` that have no owner or date

**5. Complexity**
- Flatten nested conditionals: early returns beat deep nesting
- If a function exceeds 20 lines of logic (not counting declarations), flag it for decomposition
- Guard clauses first, happy path last

**6. Type Completeness**
- For typed languages: every public interface should have explicit types
- For Python: add type hints to function signatures if missing
- Flag `any` / `object` / `dict` return types as weak — suggest a narrower type

**7. Language-Specific Idioms**
- **JavaScript/TypeScript:** prefer `const` over `let`, arrow functions for callbacks, `?.` for nullable chains
- **Python:** list comprehensions over `map`/`filter` where readable, `dataclass` over plain dicts for structured data
- **Go:** handle errors inline, never `_` an error from an important operation
- Apply the idioms of the language, not generic patterns

### Phase 3: Implement Changes

For every modification, output:

```
CHANGE: <what changed>
REASON: <which smell this fixes>
BEFORE: <original snippet>
AFTER: <new snippet>
```

### Phase 4: Score the Result

After all changes, output:

```
RESHAPE SCORE
BEFORE: X/10
AFTER:  Y/10

GAINS: <one line per improved dimension>
UNCHANGED: <anything left that could not be safely addressed>
```

Score on: naming clarity (0–3), function size (0–2), duplication (0–2), type safety (0–2), idiomatic style (0–1).

## Red Flags — Stop Immediately

- Refactoring changes what the code does — stop immediately and flag it
- You are introducing new dependencies — this is beyond scope
- Tests fail after refactoring — revert and diagnose
- The file is large enough that breaking it up requires new file structure — this needs a separate architecture pass
- The original code has no tests — add tests before refactoring

## Common Rationalizations

| Excuse | Why It's Wrong |
|--------|---|
| "The code works, so we don't need to refactor it" | Code clarity directly affects future maintenance cost. Unreadable code compounds bugs and slows onboarding. |
| "Let's refactor everything while we're at it" | Refactoring scope creep masks the original intent. Narrow changes are safer and easier to review. |
| "I'll rewrite this from scratch" | Rewriting loses edge cases. Refactoring preserves behaviour while improving clarity. |
| "Comments will explain the confusing name" | Comments rot. Names are always right because they live with the code. Rename instead. |
| "This test is too tight, let's loosen it for the refactor" | If the test breaks during a refactor, the refactor changed behaviour. Fix the refactoring, not the test. |
| "I don't need to run tests, I just renamed variables" | Refactoring at scale affects type inference, imports, and shadowing. Always run tests. |

## Quick Reference

| Phase | Core Action | Done When |
|-------|---|---|
| **Plan** | State target, smells, changes, risk level | Confirmation received (or LOW risk confirmed) |
| **Identify** | Work through code; note each smell before changing | All 7 categories checked, nothing passed over |
| **Implement** | Make one change at a time; output CHANGE/REASON/BEFORE/AFTER | All identified smells addressed |
| **Score** | BEFORE/AFTER metrics; note gains and unchanged items | Score shows improvement, tests pass |
