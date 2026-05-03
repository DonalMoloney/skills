---
name: refine-claude-md
description: Revise an existing CLAUDE.md for clarity, accuracy, and conciseness without adding new content — editing what is there, not augmenting it.
---

# Refine CLAUDE.md

## The Law
**Refining edits what exists — every sentence must become clearer, shorter, or more accurate. Adding new information belongs to improve-claude-md, not here.**

## When to Use
- CLAUDE.md has grown verbose and you want it leaner without losing anything important
- Wording is ambiguous and Claude keeps asking for clarification on things the file should answer
- A section's accuracy is in doubt after a code change but the structure is basically sound
- **Never skip when:** the file has not been reviewed since a significant refactor — drift in accuracy compounds quietly until Claude starts making wrong assumptions

## Process

### Phase 1: Read and Annotate
1. Read the target CLAUDE.md in full.
2. For every paragraph, apply three tests:
   - **Clarity test:** Could a new team member follow this instruction without asking a follow-up question?
   - **Accuracy test:** Is this still true given the current codebase? Verify commands by running them.
   - **Necessity test:** If this paragraph were deleted, would Claude miss anything important?
3. Flag each paragraph as: Pass / Reword / Shorten / Verify / Remove.

### Phase 2: Clarity Checks
Work through every flagged "Reword" item:
1. Replace vague verbs ("handle", "manage", "deal with") with specific actions ("run", "call", "write to").
2. Turn passive constructions into direct instructions ("Tests should be run" → "Run tests with `pytest -x`").
3. Break compound sentences at conjunctions — one instruction per sentence.
4. Replace relative terms ("the config folder") with absolute references ("`.claude/settings.json`").

### Phase 3: Accuracy Verification
For each flagged "Verify" item:
1. Run the documented command and confirm it exits cleanly.
   ```bash
   # Example: verify a build command still works
   npm run build 2>&1 | tail -5
   ```
   Python equivalent:
   ```bash
   python -m pytest --tb=no -q 2>&1 | tail -5
   ```
2. Check that file paths mentioned still exist:
   ```bash
   # Check a path referenced in CLAUDE.md
   ls src/api/index.ts 2>/dev/null || echo "MISSING — remove from CLAUDE.md"
   ```
3. If a command fails or a path is missing, mark it for removal or correction.

### Phase 4: Conciseness Pass
Work through every flagged "Shorten" or "Remove" item:
1. Delete any sentence that only restates what the preceding sentence already said.
2. Collapse bullet lists longer than six items — if a list needs more, split it into subsections.
3. Remove any section that scores "Remove" on the necessity test — a shorter file is a better file.
4. Merge adjacent bullets that cover the same action into one line.

Aim: every line that survives earns its place.

### Phase 5: Show Diffs and Apply
1. Present every proposed change as a before/after diff — no silent edits.
2. Group diffs by section so the user can approve or reject one section at a time.
3. After approval, apply edits using Edit — never rewrite the whole file from scratch.
4. Re-read the file after the last edit to confirm final state matches what was approved.

Example diff format:
```
### Section: Commands

Before:
  You should run the tests using the test runner configured for this project.

After:
  Run tests: `pytest -x -q`
```

## Red Flags — Stop Immediately
- You are adding a new section instead of refining existing ones — switch to improve-claude-md
- A command you are about to mark as accurate has not been verified by running it
- You are rewriting the file wholesale instead of making targeted edits
- A "Remove" candidate contains the only documentation of a non-obvious gotcha — flag it for the user before deleting

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "I'll just clean it up while adding this new bit" | Mixing refine and improve in one pass makes review impossible |
| "That sentence is a bit vague but probably fine" | Vague instructions cause Claude to ask the same question every session |
| "I didn't run the command, but it looks right" | Unverified commands silently break workflows for weeks |
| "Removing that section might delete something useful" | The necessity test exists for this — if it passes, keep it; if not, cut it |
| "One big rewrite is faster than targeted edits" | Wholesale rewrites discard nuance and erase intentional phrasing |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1. Read and Annotate | Flag each paragraph: Pass / Reword / Shorten / Verify / Remove | Every paragraph has a disposition |
| 2. Clarity | Replace vague wording with specific instructions | No sentence survives the clarity test with a fail |
| 3. Accuracy | Run commands, check paths | Every surviving instruction verified against actual code |
| 4. Conciseness | Delete redundancy, collapse long lists | File is shorter; nothing important is missing |
| 5. Diffs | Show before/after per section, apply after approval | File matches approved diffs; re-read confirms final state |
