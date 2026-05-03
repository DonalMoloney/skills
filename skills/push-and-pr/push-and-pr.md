---
name: push-and-pr
description: Commit local changes, push to origin, and open a pull request with a structured description.
---

# Push and PR

## The Law
**Never push directly to the default branch — always create a feature branch, get explicit confirmation before the push, and open a PR with a complete description so reviewers have the context they need.**

## When to Use
- A logical unit of work is complete and ready for review
- You need to share changes with teammates or trigger CI
- The current branch has commits that do not yet exist on the remote
- **Never skip when:** the change is "small" (small unreviewed changes accumulate into large unmaintained branches), or you think you can merge without review (that decision belongs to a reviewer, not the author)

## Process

### Phase 1: Confirm Branch and State
1. Run `git branch --show-current` — if you are on `main`, `master`, or another protected branch, create a feature branch before proceeding:
   ```bash
   git checkout -b feat/your-feature-name
   ```
2. Run `git status` and `git diff HEAD` — confirm nothing unintended is present.
3. Run `git log origin/$(git branch --show-current)..HEAD 2>/dev/null || git log --oneline -10` to understand what will be pushed.

### Phase 2: Commit (if needed)
1. If uncommitted changes exist, follow the clean-commit skill to produce a proper commit.
2. Do not proceed with a dirty working tree — stash or commit first.

### Phase 3: Confirmation Gate
1. Print a summary of what will happen:
   - Branch name
   - Number of commits to push
   - Target remote (`origin`)
2. Pause and ask for explicit confirmation before any network operation. This is the gate — do not skip it.
   ```
   Ready to push branch 'feat/login-rate-limit' (2 commits) to origin.
   Proceed? [y/N]
   ```
3. If the answer is anything other than an affirmative, stop and report what was staged.

### Phase 4: Push
1. Push the branch with tracking set:
   ```bash
   git push -u origin "$(git branch --show-current)"
   ```
2. Read the output. If the remote rejects the push (non-fast-forward), do not force-push — investigate why and rebase if appropriate:
   ```bash
   git fetch origin
   git rebase origin/main
   ```
3. Never use `--force` or `--force-with-lease` without the user's explicit instruction.

### Phase 5: Open the Pull Request
1. Build the PR body using the template below.
2. Create the PR via `gh`:
   ```bash
   gh pr create \
     --title "Add rate-limit middleware to login endpoint" \
     --body "$(cat <<'EOF'
   ## Summary
   - Added per-IP rate limiting to the login endpoint
   - Configurable via RATE_LIMIT_MAX env var (default 10/min)
   - Returns 429 with Retry-After header when limit is exceeded

   ## Test plan
   - [ ] Unit tests pass: `pytest tests/test_login.py`
   - [ ] Manual test: hammer endpoint, confirm 429 after threshold
   - [ ] Env var override tested locally

   ## Notes
   Requires RATE_LIMIT_MAX to be set in staging environment.
   EOF
   )"
   ```
3. Confirm the PR URL is printed and accessible.

### PR Body Template
```
## Summary
- <bullet: what changed>
- <bullet: why it changed>
- <bullet: notable trade-offs or dependencies>

## Test plan
- [ ] <command or manual step>
- [ ] <command or manual step>

## Notes
<anything a reviewer needs to know that is not in the code>
```

## Red Flags — Stop Immediately
- You are about to push to `main`, `master`, `develop`, or any other protected branch directly
- The confirmation gate was skipped or bypassed
- `git status` shows uncommitted changes going into the push
- The PR title is a placeholder: `fix`, `wip`, `update`, `changes`
- `--force` is about to be used without explicit user approval
- The PR body is empty or missing a test plan

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "It's a tiny fix, no PR needed" | Size does not determine review need; risk does |
| "I'll update the PR description later" | Reviewers act on what is there now; an empty description wastes their time |
| "Force-push is fine on my own branch" | It rewrites history others may have already fetched |
| "Skipping the confirmation gate saves a step" | The gate exists to catch pushes to wrong branches |
| "CI will catch any issues" | CI is a safety net, not a substitute for a readable PR |
| "I'm the only reviewer anyway" | Self-merging without a PR breaks the audit trail |
| "The PR template is too long" | Trim it once — do not skip it |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1. Confirm branch | Check current branch; create feature branch if on main | On a non-protected branch |
| 2. Commit | Stage and commit all changes cleanly | Working tree is clean |
| 3. Gate | Print summary, wait for explicit yes | User confirmed push |
| 4. Push | `git push -u origin <branch>` | Remote accepts push, no errors |
| 5. PR | `gh pr create` with full template | PR URL returned and accessible |
