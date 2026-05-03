---
name: request-review
description: Use after completing a task, finishing a major feature, or before merging — dispatches a code-reviewer subagent with precisely scoped context so the review focuses on the work, not the session history
---

# Request Review

## The Law
**Every significant change must go through a code review before it is considered done — dispatch the reviewer with exact git SHAs and a clear scope statement, never with a vague "look at my recent work."**

## When to Use
- After each task in a multi-task plan (catch problems before they compound)
- After landing a major feature
- Before merging any branch to main
- **Never skip when:** the change touches shared infrastructure, auth, data persistence, or public interfaces — these are precisely the areas where "it looks fine to me" fails most often

Optional but valuable:
- When progress has stalled and a second perspective might unlock it
- After untangling a gnarly bug fix, to confirm the fix is complete

## Process

### Phase 1: Gather Review Coordinates

1. Determine the commit range to review:
   ```bash
   # Last commit only
   BASE_SHA=$(git rev-parse HEAD~1)
   HEAD_SHA=$(git rev-parse HEAD)

   # Since diverging from the remote main
   BASE_SHA=$(git merge-base HEAD origin/main)
   HEAD_SHA=$(git rev-parse HEAD)

   # Since a specific task commit
   BASE_SHA=$(git log --oneline | grep "Task 1 complete" | awk '{print $1}')
   HEAD_SHA=$(git rev-parse HEAD)
   ```

   Python equivalent using `subprocess`:
   ```python
   import subprocess

   def get_sha(ref: str) -> str:
       return subprocess.check_output(
           ["git", "rev-parse", ref], text=True
       ).strip()

   base_sha = get_sha("HEAD~1")
   head_sha = get_sha("HEAD")
   ```

2. Confirm the diff covers exactly what was implemented:
   ```bash
   git diff --stat "$BASE_SHA".."$HEAD_SHA"
   ```

### Phase 2: Build the Context Bundle

Assemble the four pieces the reviewer needs — keep each one tight:

1. **What was built** — one or two sentences describing the implementation, not the process
2. **What it should do** — the requirement or task description it satisfies
3. **The commit range** — `BASE_SHA` and `HEAD_SHA`
4. **A brief description** — a single line suitable for a review header

Avoid including session history, exploratory attempts, or reasoning steps — the reviewer evaluates the work product, not the journey.

### Phase 3: Dispatch the Reviewer Subagent

Use the Task tool with `superpowers:code-reviewer` and populate these fields:

| Field | Content |
|-------|---------|
| `WHAT_WAS_IMPLEMENTED` | What you built (from Phase 2, item 1) |
| `PLAN_OR_REQUIREMENTS` | What it should do (from Phase 2, item 2) |
| `BASE_SHA` | Starting commit hash |
| `HEAD_SHA` | Ending commit hash |
| `DESCRIPTION` | Single-line review header |

### Phase 4: Act on Feedback

Triage the returned issues by severity:

1. **Critical** — fix immediately before doing anything else
2. **Important** — fix before moving to the next task or merging
3. **Minor** — log for cleanup; do not block on these

If the reviewer is wrong about something:
- State the technical objection with evidence (test output, code reference)
- Do not simply defer — reviewers can lack full context
- Escalate to the user if there is an architectural disagreement

## Red Flags — Stop Immediately

- Skipping review because the change is "just a refactor" or "just tests"
- Dispatching the reviewer without `BASE_SHA` and `HEAD_SHA` — vague scope produces vague feedback
- Moving to the next task while a Critical issue remains open
- Accepting reviewer feedback without checking whether it applies to this codebase

## Common Rationalizations

| Excuse | Why It's Wrong |
|--------|----------------|
| "It's too simple to need review" | Simple changes are fast to review; skipping builds a habit of skipping |
| "I'll review it myself before merging" | Self-review misses the same blindspots that created the issue |
| "The reviewer might slow me down" | A post-merge fix is slower than pre-merge feedback |
| "The tests cover it" | Tests verify behaviour; review catches structure, security, and intent gaps |
| "I know this code better than any reviewer" | That familiarity is exactly why a fresh perspective matters |

## Quick Reference

| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1 — Coordinates | Capture BASE_SHA and HEAD_SHA | Diff verified against intent |
| 2 — Context bundle | Write scope, requirements, description | Four fields populated, no session history |
| 3 — Dispatch | Launch code-reviewer subagent | Subagent returns structured feedback |
| 4 — Act | Triage and fix by severity | All Critical and Important issues resolved |
