---
name: improve-claude-md
description: Audit every CLAUDE.md in the repo against quality criteria, produce a scored report, then apply targeted improvements after user approval.
---

# Improve CLAUDE.md

## The Law
**Always show the scored quality report and get explicit approval before changing a single line of any CLAUDE.md file.**

## When to Use
- Starting work on an unfamiliar codebase and CLAUDE.md feels thin or stale
- A command documented in CLAUDE.md no longer works
- Adding a new package, tool, or workflow that future sessions need to know about
- **Never skip when:** the project has not been touched in a while — outdated instructions actively mislead Claude and cost tokens correcting mistakes

## Process

### Phase 1: Discovery
1. Find every CLAUDE.md variant in the repository:
   ```bash
   find . -name "CLAUDE.md" -o -name ".claude.md" -o -name ".claude.local.md" 2>/dev/null | sort
   ```
2. Note each file's location and its role:

| Location | Role |
|----------|------|
| `./CLAUDE.md` | Project root — shared with the whole team via git |
| `./.claude.local.md` | Personal overrides — gitignored, not committed |
| `~/.claude/CLAUDE.md` | Global defaults — apply to every project for this user |
| `./packages/*/CLAUDE.md` | Package-level context in monorepos |

### Phase 2: Quality Assessment
Score each file against six criteria. Maximum score: 100.

**Structure Checklist (run for each file):**
- [ ] Build / test / lint commands present and copy-paste ready
- [ ] Directory layout or architecture described
- [ ] Entry points and key config files named
- [ ] Environment variables and required tooling listed
- [ ] Non-obvious gotchas and project quirks captured
- [ ] No content that restates what the code makes obvious

| Criterion | Max | Failing Signals |
|-----------|-----|-----------------|
| Commands and workflows | 20 | No build/test command, or command no longer works |
| Architecture clarity | 20 | No directory map, unclear module boundaries |
| Non-obvious patterns | 15 | Missing gotchas, quirks, or known traps |
| Conciseness | 15 | Walls of prose, generic best-practice lists |
| Currency | 15 | References removed packages, old paths |
| Actionability | 15 | Vague phrases like "configure appropriately" |

Grades: A = 90–100, B = 70–89, C = 50–69, D = 30–49, F = 0–29

### Phase 3: Quality Report
Produce the full report before touching any file:

```
## CLAUDE.md Quality Report

### Summary
- Files found: 2
- Average score: 61/100
- Files needing update: 2

### ./CLAUDE.md — Score: 58/100 (Grade: C)

| Criterion | Score | Notes |
|-----------|-------|-------|
| Commands | 10/20 | Test command present; build command missing |
| Architecture | 14/20 | Good module overview; no entry-point file named |
| Non-obvious patterns | 8/15 | Auth quirk documented; DB migration trap missing |
| Conciseness | 12/15 | One redundant section |
| Currency | 8/15 | References src/old-api/ which was removed |
| Actionability | 6/15 | "Set up your environment" gives no actual steps |

Issues:
- Build command absent — Claude must guess or ask every session
- Path src/old-api/ no longer exists

Recommended additions:
- npm run build under a Commands section
- Remove or correct the stale src/old-api/ reference
- Add the missing DB migration gotcha
```

### Phase 4: Propose Changes
For each issue, show exactly what will be added or removed as a diff:

```
### Update: ./CLAUDE.md

Why: Build command was absent, causing repeated clarification requests.

+ ## Commands
+
+ ```bash
+ npm run build      # compile TypeScript to dist/
+ npm test           # run Jest test suite
+ npm run lint       # ESLint + Prettier check
+ ```
```

Rules for what to include:
- Commands and workflows discovered by reading the code — copy-paste ready
- Gotchas found in commit history, test failures, or CI config
- Package relationships that are not obvious from directory names

Rules for what to exclude:
- Generic advice already in every project ("write clean code")
- One-off fixes unlikely to recur
- Anything obvious from reading the file structure

Python equivalent command documentation format:
```python
# Commands section example for a Python project
# pytest -x -q          run tests; stop on first failure
# ruff check .          lint the codebase
# ruff format .         format all Python files
# python -m build       build the distribution package
```

### Phase 5: Apply Updates
1. Confirm the user has approved each proposed change.
2. Apply with Edit — preserve the existing structure; insert, do not overwrite.
3. After each file update, re-read the file to confirm the edit landed correctly.
4. Remind the user: pressing `#` during any session lets Claude auto-incorporate fresh learnings into CLAUDE.md on the fly.

## Red Flags — Stop Immediately
- You are about to edit a CLAUDE.md without showing the quality report first
- The proposed addition restates something the code already makes obvious
- You are adding generic best-practice content that applies to every project equally
- A stale command is being left in because "it might still work" — verify or remove it
- You are editing `~/.claude/CLAUDE.md` (global) when only the project file needs changing

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "The file looks fine at a glance" | A formal score catches hidden gaps; gut feel misses them |
| "I'll add a quick note without showing a diff" | Unseen edits erode trust and introduce formatting drift |
| "Generic advice is still useful" | It dilutes signal and trains Claude to ignore CLAUDE.md |
| "The old path might come back" | Dead paths in CLAUDE.md cause hallucinations — remove them now |
| "Local override handles it, no need to update root" | Local files are personal; the root file serves the whole team |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1. Discovery | `find` all CLAUDE.md variants | Full file list with locations |
| 2. Assessment | Score each file against six criteria | Numeric scores and grade per file |
| 3. Report | Print scored report | User has seen all scores and issues |
| 4. Propose | Show diffs for each fix | User has approved or rejected each change |
| 5. Apply | Edit files, verify each write | Files updated, no stale content remains |
