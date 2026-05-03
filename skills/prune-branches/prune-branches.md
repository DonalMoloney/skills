---
name: prune-branches
description: Find and remove local branches whose remote counterparts have been deleted, with a dry-run preview before any deletion.
---

# Delete Gone Branches

## The Law
**Always run a dry-run first — print every branch and worktree that will be removed, get confirmation, then delete. Never bulk-delete without a preview.**

## When to Use
- After a sprint or release cycle when many feature branches have been merged and deleted on the remote
- When `git branch -v` shows several `[gone]` entries cluttering the list
- Before starting new work to ensure a clean local branch namespace
- **Never skip when:** you have worktrees active — gone branches with worktrees need worktree removal first, which the dry-run surfaces

## Process

### Phase 1: Fetch and Sync Remote State
1. Update remote-tracking references so `[gone]` status is current:
   ```bash
   git fetch --prune
   ```
2. Confirm the fetch succeeded before continuing — a network error means the gone list is stale.

### Phase 2: Dry Run — Identify What Will Be Deleted
1. List all local branches with their tracking status:
   ```bash
   git branch -v
   ```
2. Extract only the branches marked `[gone]`:
   ```bash
   git branch -v | grep '\[gone\]'
   ```
3. List all active worktrees to find any that are tied to gone branches:
   ```bash
   git worktree list
   ```
4. Print the complete removal plan — branches to delete and worktrees to remove — before touching anything:
   ```
   Dry run complete. Will remove:
     Worktrees:  /path/to/worktree  [feat/login-rate-limit]
     Branches:   feat/login-rate-limit
                 fix/header-null
                 chore/bump-deps
   Proceed? [y/N]
   ```
5. If the list is empty, report "No gone branches found — nothing to do" and stop.

### Phase 3: Confirmation Gate
1. Wait for explicit confirmation before any deletion.
2. If the user wants to selectively delete, handle individual branches rather than the full batch.
3. A non-affirmative answer cancels the entire operation — do not delete any subset silently.

### Phase 4: Remove Worktrees First
1. For every gone branch that has an associated worktree, remove the worktree before attempting branch deletion:
   ```bash
   git worktree remove --force "/path/to/worktree"
   ```
2. Verify the worktree is gone: `git worktree list` should no longer show it.
3. If worktree removal fails, stop — do not attempt the branch deletion for that entry.

### Phase 5: Delete Gone Branches
1. Run the full cleanup for all confirmed gone branches:
   ```bash
   git branch -v | grep '\[gone\]' | sed 's/^[+* ]*//' | awk '{print $1}' | while read branch; do
     echo "Deleting: $branch"
     git branch -D "$branch"
   done
   ```
2. Verify with a final `git branch -v` — no `[gone]` entries should remain.
3. Report a count: "Removed 3 branches and 1 worktree."

### Phase 6: Confirm Clean State
1. Run `git branch -v` one final time and confirm no gone entries remain.
2. Run `git worktree list` and confirm no orphaned worktrees remain.

## Red Flags — Stop Immediately
- `git fetch --prune` produced an error — the gone list is unreliable
- A branch in the gone list is your current checkout (`*` prefix in `git branch -v`)
- A worktree path shown in the list is also the main repository root
- The user has not confirmed the dry-run output before deletion begins
- The script would delete `main`, `master`, `develop`, or any explicitly protected branch name

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "I know which branches are gone — skip the dry run" | Memory is not a substitute for a printed list you can verify |
| "The worktree is empty anyway" | Git does not know that; force-removing may lose uncommitted work |
| "It's faster to just delete everything in one command" | One accidental branch deletion can mean lost unmerged commits |
| "I'll just re-fetch the branch if I need it" | Only possible if the remote hasn't garbage-collected it |
| "The fetch --prune step is optional" | Without it, your gone list reflects yesterday's remote, not today's |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1. Fetch | `git fetch --prune` | Remote refs are current |
| 2. Dry run | List gone branches and affected worktrees | Full removal plan printed |
| 3. Gate | Print plan, wait for yes | User confirmed deletion |
| 4. Worktrees | `git worktree remove --force` per entry | No orphaned worktrees |
| 5. Delete | `git branch -D` for each gone branch | `git branch -v` shows no `[gone]` |
| 6. Verify | Final `git branch -v` + `git worktree list` | Clean state confirmed |
