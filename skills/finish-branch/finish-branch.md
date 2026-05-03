---
name: finish-branch
description: Use when your branch is ready to merge — final checklist before PR, verify tests pass, clean up commits, confirm remote is pushed, then open the PR.
---

# Finish Branch

## The Law
**Before opening a PR, verify: all tests pass, commits are clean, code is pushed, and branch is up to date with main. A PR that fails these basics wastes reviewer time.**

## When to Use
- Your feature branch is feature-complete and tested
- All local work is committed
- You're ready to ask for code review
- **Never skip when:** the work touches shared code — reviewers depend on clean PRs

## Process

### Phase 1: Verify Tests Pass
1. Run the full test suite:
   ```bash
   npm test  # or pytest, go test, etc.
   ```
2. Fix any failures before proceeding
3. Run lint and type checks:
   ```bash
   npm run lint
   npm run type-check
   ```

### Phase 2: Verify Branch is Up to Date
1. Fetch latest main:
   ```bash
   git fetch origin main
   ```
2. Rebase on main if needed:
   ```bash
   git rebase origin/main
   ```
3. If conflicts, resolve and test again
4. Push rebased branch:
   ```bash
   git push origin <branch> --force-with-lease
   ```

### Phase 3: Review Commits
1. Review commit history:
   ```bash
   git log origin/main..HEAD
   ```
2. Each commit should have:
   - Clear, imperative message (not a summary)
   - Single logical purpose
   - Tests for that purpose
3. Squash or reorder commits if needed

### Phase 4: Create the PR
1. Push to remote if not already pushed:
   ```bash
   git push origin <branch>
   ```
2. Open PR on GitHub/GitLab
3. Fill in PR template:
   - Concise title (under 70 chars)
   - Summary of what changed and why
   - Relevant issue links
   - Test instructions if needed
4. Request reviewers
5. Link any related issues or PRs

### Phase 5: Monitor for Feedback
1. Watch for CI checks to pass
2. Respond promptly to reviewer comments
3. Fix requested changes with new commits (don't squash)
4. Re-request review after changes

## Red Flags — Stop Immediately

- Tests are failing — fix before opening PR
- Branch is out of sync with main — rebase first
- Commits have placeholder messages — rewrite them
- The PR title is vague — clarify it
- You don't understand why you made a change — add a comment
- CI is failing on the branch — investigate before asking for review

## Common Rationalizations

| Excuse | Why It's Wrong |
|--------|---|
| "I'll fix the tests in code review" | Reviewers don't fix code. They review it. Fix tests before PR. |
| "The branch is old, but the code is still good" | Old branches are out of sync. Rebase and test to be sure. |
| "My commits are messy, but reviewers can figure it out" | Messy commits make review harder. Clean them up. |
| "I'll just open the PR and let CI tell me if something is wrong" | Don't. Run tests locally first. CI is a verification step, not a first test. |

## Quick Reference

| Phase | Core Action | Done When |
|-------|---|---|
| **Tests** | Run full test suite; fix failures | All tests pass, lint passes, types pass |
| **Update** | Fetch main; rebase if needed; push | Branch is up to date with main |
| **Review** | Check commits are clean and purposeful | Each commit has clear message and purpose |
| **Open** | Push branch; fill PR template; request review | PR is open with clear title and description |
| **Monitor** | Watch CI; respond to review comments | All feedback addressed; ready to merge |
