---
name: git-worktree
description: Use when starting feature work that must stay isolated from the current workspace, or before executing implementation plans — creates a linked worktree with safety checks and a verified baseline
---

# Git Worktree

## The Law
**Every project-local worktree directory must be confirmed ignored by git before any worktree is created inside it — unignored worktrees pollute history and must be fixed on the spot.**

## When to Use
- Starting a feature that must not disturb the current working branch
- Executing a plan that requires an isolated environment
- Running multiple branches in parallel without repeated `git stash` / `git switch` cycles
- **Never skip when:** a plan or parallel-agent workflow is about to begin — isolation prevents cross-branch contamination that is hard to undo

## Process

### Phase 1: Choose the Worktree Directory

Work through these checks in order and stop at the first match:

1. Look for an existing directory:
   ```bash
   ls -d .worktrees 2>/dev/null
   ls -d worktrees  2>/dev/null
   ```
   If both exist, `.worktrees` wins. If one exists, use it.

2. Check `CLAUDE.md` for a declared preference:
   ```bash
   grep -i "worktree.*dir" CLAUDE.md 2>/dev/null
   ```
   Use whatever it specifies without asking.

3. If neither check yielded a result, ask:
   ```
   No worktree directory found. Where should new worktrees go?

   1. .worktrees/  (project-local, hidden from directory listings)
   2. ~/.config/worktrees/<project-name>/  (global, outside the repo)

   Which would you prefer?
   ```

### Phase 2: Verify Gitignore Coverage (project-local directories only)

Skip this phase if the chosen directory is outside the repository (e.g., `~/.config/...`).

1. Test whether git ignores the directory:
   ```bash
   git check-ignore -q .worktrees 2>/dev/null
   # or
   git check-ignore -q worktrees 2>/dev/null
   ```

2. If the directory is NOT ignored:
   - Add it to `.gitignore` immediately
   - Commit the change before creating the worktree:
     ```bash
     echo ".worktrees/" >> .gitignore
     git add .gitignore
     git commit -m "chore: ignore worktree directory"
     ```

### Phase 3: Create the Worktree

1. Capture the project name:
   ```bash
   project=$(basename "$(git rev-parse --show-toplevel)")
   ```

2. Build the full path and create the worktree on a new branch:
   ```bash
   # Project-local example
   git worktree add .worktrees/<branch-name> -b <branch-name>

   # Global example
   git worktree add ~/.config/worktrees/<project>/<branch-name> -b <branch-name>
   ```

3. Switch into the new worktree directory.

### Phase 4: Install Dependencies

Auto-detect from project files and run the appropriate setup — do not hardcode a single tool:

```bash
# Node.js
[ -f package.json ]      && npm install

# Python (try both common setups)
[ -f requirements.txt ]  && pip install -r requirements.txt
[ -f pyproject.toml ]    && poetry install

# Rust
[ -f Cargo.toml ]        && cargo build

# Go
[ -f go.mod ]            && go mod download
```

Python example if the project uses a virtual environment:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
# or
pip install -e ".[dev]"
```

### Phase 5: Verify a Clean Baseline

Run the full test suite before any implementation begins:

```bash
# Examples — use whichever matches the project
npm test
pytest
cargo test
go test ./...
```

Python with verbose output:
```bash
pytest -v --tb=short
```

- If tests fail: report every failure and ask whether to investigate or proceed anyway.
- If tests pass: report the count and confirm the worktree is ready.

### Phase 6: Report Ready

```
Worktree ready at <full-path>
Branch: <branch-name>
Tests: <N> passing, 0 failures
Ready to begin <feature-name>
```

## Red Flags — Stop Immediately

- About to create a project-local worktree without running `git check-ignore` first
- Directory is not ignored and the `.gitignore` fix is being skipped
- Assuming a directory location without checking existing dirs or `CLAUDE.md`
- Proceeding with implementation when the baseline test run has failures
- Hardcoding `npm install` when the project is Python, Rust, or Go

## Common Rationalizations

| Excuse | Why It's Wrong |
|--------|----------------|
| "The directory probably ends up in .gitignore already" | Probably is not verified — check with `git check-ignore` |
| "I know where the worktrees should go" | Conventions belong in the project, not in assumptions |
| "The tests were passing yesterday" | Yesterday's state is not today's baseline |
| "Dependency install takes too long, I'll skip it" | A broken dependency silently corrupts the entire baseline |
| "The feature is urgent, ignore the failures" | Urgent work needs a known-good starting point more than routine work does |

## Quick Reference

| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1 — Choose directory | existing > CLAUDE.md > ask | Directory path confirmed |
| 2 — Gitignore check | `git check-ignore` + fix if needed | Directory is git-ignored |
| 3 — Create worktree | `git worktree add` on new branch | Worktree directory exists |
| 4 — Dependencies | Auto-detect and install | No install errors |
| 5 — Baseline | Run full test suite | Zero failures reported |
| 6 — Report | State path, branch, test count | User knows the worktree is ready |
