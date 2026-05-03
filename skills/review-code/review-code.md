---
name: review-code
description: Invoke before committing or opening a PR to check code against project guidelines, catch bugs, and confirm CLAUDE.md compliance.
---

# Review Code

## The Law
**No code ships without a review pass against project guidelines and a confidence-scored issue list — surface only what genuinely matters, never quantity over quality.**

## When to Use
- A feature implementation is complete and ready to commit
- The assistant has just written new code and should self-check before declaring done
- A PR is about to be opened and needs a final diff review
- **Never skip when:** the diff touches error handling, authentication, data mutations, or any code path called by a public API

## Process

### Phase 1: Establish Review Scope
1. Determine what to review
   - Default: unstaged changes via `git diff`
   - If the user specifies files or a commit range, use that instead
2. Open CLAUDE.md (or equivalent project rules file) and read every rule before writing a single finding
   - Note import patterns, naming conventions, framework constraints, error handling rules, testing requirements
3. List the files under review so the output is traceable

### Phase 2: Check CLAUDE.md Compliance
Walk the diff line by line against every explicit rule in CLAUDE.md:

1. Import patterns — are imports sourced from the right modules?
2. Framework conventions — are the right APIs, decorators, or hooks used?
3. Naming conventions — functions, variables, classes, files
4. Error handling rules — does the diff follow the project's required logging and propagation patterns?
5. Testing requirements — does new logic have corresponding tests?
6. Platform compatibility notes — are any platform-specific guards required?

Flag each violation with its exact CLAUDE.md rule reference.

### Phase 3: Bug Detection
Examine the logic for defects that will affect runtime behaviour:

1. Logic errors — does the code do what the author intended?
2. Null and undefined handling — are all nullable paths guarded?
3. Race conditions — are concurrent mutations or shared state accesses safe?
4. Resource management — are files, connections, and locks released on all exit paths?
5. Security — is user input validated before use? Are secrets handled safely?
6. Performance — are there O(n²) loops, repeated I/O in tight paths, or unbounded allocations?

Python examples of common bugs to check for:
```python
# Race condition: read-modify-write without a lock
counter += 1   # not atomic in threaded code

# Resource leak: file not closed on exception
f = open("data.csv")
data = f.read()   # DEFECT: f never closed if read() raises
f.close()

# Unchecked None from a dict lookup
user = users.get(user_id)
print(user.name)   # DEFECT: AttributeError if user_id absent
```

TypeScript equivalent:
```typescript
// Unchecked null from a map lookup
const user = users.get(userId);
console.log(user.name);   // DEFECT: TypeError if userId absent — use optional chaining with a guard
```

### Phase 4: Code Quality
Look for structural issues that compound over time:

1. Duplication — is the same logic copy-pasted rather than extracted?
2. Missing critical error handling — not style, but paths where an unhandled error will crash a user session
3. Accessibility — are interactive elements keyboard-navigable and screen-reader labelled?
4. Test coverage gaps — does new branching logic lack a test for the unhappy path?

### Phase 5: Score and Report
Rate every candidate issue from 0 to 100:

- **0–25**: Likely a false positive or a pre-existing issue outside the diff
- **26–50**: Minor nitpick not backed by a CLAUDE.md rule
- **51–75**: Valid but low-impact issue
- **76–90**: Important issue that needs attention before merge
- **91–100**: Critical defect or explicit CLAUDE.md violation

**Report only issues scoring 80 or above.**

Format each finding:
- Confidence score and severity label (Critical: 91–100, Important: 80–90)
- File path and line number
- Which CLAUDE.md rule it breaks, or which bug class it falls into
- A concrete, specific fix — not just "improve this"

If no issues score 80 or above, state clearly that the code meets project standards and give a one-sentence summary of what was checked.

## Red Flags — Stop Immediately
- Diff touches authentication or authorization logic with no security check performed
- New code paths have no error handling at all
- A rule in CLAUDE.md is directly contradicted by the diff
- Production code references a test double, mock, or hardcoded credential
- A function with side effects (writes, sends, deletes) has no corresponding test

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "It's a small change, review is overkill" | Small changes introduce the majority of production bugs — size is not a proxy for safety |
| "I'll fix style issues later" | Style drift becomes the norm; CLAUDE.md rules exist to prevent it compounding |
| "The tests pass, so it's fine" | Tests check what was anticipated — a review catches what was not |
| "I know this codebase, I don't need to check the rules" | Rules change; checking CLAUDE.md takes 30 seconds and prevents a rework cycle |
| "Low-confidence issues aren't worth reporting" | Correct — the 80-threshold exists precisely to enforce this; report only what matters |
| "I reviewed it mentally while writing" | Mental reviews miss line-by-line issues; a structured pass catches what familiarity hides |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1. Scope | Confirm what is under review; read CLAUDE.md | File list and rules in hand |
| 2. Compliance | Check every CLAUDE.md rule against the diff | All rules evaluated, violations flagged |
| 3. Bugs | Logic errors, nulls, races, resources, security, performance | All six bug classes checked |
| 4. Quality | Duplication, missing error handling, accessibility, test gaps | Structural issues assessed |
| 5. Report | Score every issue; report only ≥80 | Scored list delivered or clean-bill confirmed |
