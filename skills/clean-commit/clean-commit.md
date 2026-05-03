---
name: clean-commit
description: Use before committing — stage specific files (never -A), write a clear message with purpose (not a summary), add co-author line, and verify the commit. Safety-first workflow.
---

# Clean Commit

## The Law
**Every commit must have a purpose — one cohesive reason that explains why these changes belong together. If you need "and" or "also" in the message, split into two commits.**

## When to Use
- Files are staged and ready to commit
- You have tested the changes locally
- The commit message explains the reason for change, not just what changed
- **Never skip when:** committing to shared branches — bad commits compound

## Process

### Phase 1: Stage Specific Files
1. Review what will be committed:
   ```bash
   git diff --cached
   ```
2. Stage files by name (never `git add -A` or `git add .`):
   ```bash
   git add path/to/file1.js path/to/file2.js
   ```
3. Verify staged files:
   ```bash
   git status
   ```

### Phase 2: Write the Commit Message
1. Use imperative present tense: "fix bug" not "fixed bug"
2. Lead with the purpose (why), not the what:
   - ✅ Good: `fix: prevent null pointer in user lookup by adding null check`
   - ❌ Bad: `fixed stuff`, `changes`, `WIP`, `temp`
3. If the message requires explanation, add a body:
   ```
   fix: prevent null pointer in user lookup

   The query does not guard against null userId, causing TypeError
   when unauthenticated requests hit /profile. Added null check
   before database query.
   ```
4. Keep the first line under 70 characters

### Phase 3: Add Co-Author Line
1. Add this line at the very end:
   ```
   Co-Authored-By: <Name> <email@example.com>
   ```

### Phase 4: Commit
1. Create the commit:
   ```bash
   git commit -m "message here"
   ```
2. Verify the commit:
   ```bash
   git log -1 -p
   ```

## Red Flags — Stop Immediately

- Staged files include unrelated changes — unstage and split into two commits
- The message is a placeholder — write a clear message or don't commit yet
- You used `git add -A` or `git add .` — you might have staged secrets or build artifacts
- The commit is too large to review in 5 minutes — break it up
- You don't understand why you're making these changes — stop and clarify first

## Common Rationalizations

| Excuse | Why It's Wrong |
|--------|---|
| "I'll clean up commits later" | You won't. Bad commits live forever in the history. |
| "The CI will catch any issues" | Tests are not the same as review. Clean commits prevent review friction. |
| "I need to commit unrelated changes together" | You don't. Every change has a reason. Give each one its own commit. |
| "Staging by name is slow" | Choosing files by name takes 30 seconds. Debugging bad commits takes hours. |
| "The message is obvious from the code diff" | Diff shows what changed. Commit message explains why. Future-you needs both. |

## Quick Reference

| Phase | Core Action | Done When |
|-------|---|---|
| **Stage** | Add files by name; verify with `git status` | Staged files are exact and correct |
| **Message** | Write purpose (why), not summary (what); keep under 70 chars | Message explains the reason clearly |
| **Co-Author** | Add co-author line at end | Co-author is present (if applicable) |
| **Commit** | Run `git commit`, verify with `git log -1 -p` | Commit is created and verified |
