---
name: check-comments
description: Use to review code comments for accuracy and staleness — verify comments explain why, not what; flag outdated comments; suggest removals for obvious code.
---

# Check Comments

## The Law
**Comments explain WHY code exists, not WHAT it does. If the code says what it does, a comment that repeats that is noise. If a comment is outdated, it's worse than no comment.**

## When to Use
- Reviewing a PR that adds comments
- Code has many comments and you're unsure if they're valuable
- Maintenance phase — cleaning up old code
- **Never skip when:** inherited code has years of comments — stale comments cause bugs

## Process

### Phase 1: Identify Comment Types
1. Scan all comments in the changed code
2. Categorize each:
   - **Why comments** — explain the reason (valuable)
   - **What comments** — explain what code does (noise if the code is clear)
   - **TODO/FIXME** — mark incomplete work (flag if no owner/date)
   - **Docstrings** — API documentation (necessary)

### Phase 2: Check Accuracy
1. For each comment, verify:
   - Does it match what the code actually does?
   - Is it still true after recent changes?
   - Would someone understand this 6 months from now?
2. Flag inaccurate or outdated comments

### Phase 3: Remove Obvious Noise
1. **What comments** are noise:
   - `// increment i` on `i++` — obvious
   - `// return result` on `return result` — obvious
   - `// check if valid` on `if (isValid)` — obvious
2. Suggest removal of these

### Phase 4: Strengthen Weak Comments
1. `// special case` → explain what makes it special
2. `// hack for X` → explain why the hack exists and when to remove it
3. `// TODO` → add owner name and date (or delete if old)

### Phase 5: Flag Staleness
1. Find comments that reference old code:
   ```python
   # This used to call the deprecated API before we refactored
   ```
2. Find TODO/FIXME without ownership:
   ```javascript
   // TODO: fix this later
   ```
3. Suggest removal or update

### Phase 6: Scorecard
Output:

```
COMMENT REVIEW
Why comments (good): X
What comments (noise): X
Outdated/stale: X

Recommendations:
- Remove X "what" comments (obvious from code)
- Update X comments (inaccurate)
- Remove X TODOs (no owner/date)
```

## Red Flags — Stop Immediately

- Comments are outdated compared to the code — flag for update or removal
- Comments explain what the code does word-for-word — remove them
- TODOs exist without owner or date — flag them
- Docstrings are missing on public APIs — add them
- Comments describe removed code — remove the comment

## Common Rationalizations

| Excuse | Why It's Wrong |
|--------|---|
| "More comments are always better" | Outdated comments are worse than no comments. Quality > quantity. |
| "This code is complex, it needs comments" | Complex code needs better names and refactoring, not comments. |
| "We'll clean up comments later" | You won't. Remove noise now. |
| "I'll keep this comment in case I forget" | That's what commit messages are for. Comments are for explaining why. |

## Quick Reference

| Phase | Core Action | Done When |
|-------|---|---|
| **Identify** | Categorize comments: why / what / TODO / docstring | All comments are categorized |
| **Accuracy** | Verify each comment matches current code | Inaccurate comments are flagged |
| **Remove** | Delete "what" comments (obvious from code) | Noise is eliminated |
| **Strengthen** | Improve weak "why" comments with detail | Comments explain the reason clearly |
| **Stale** | Flag outdated and unowned TODOs | Staleness is identified |
| **Scorecard** | Output review with recommendations | Recommendations are explicit |
