---
name: execute-plan
description: Use when you have a written implementation plan and need to execute it task-by-task in the current session with review checkpoints. Load, review, execute, verify, and hand off to branch-finish.
---

# Execute Plan

## The Law
**Follow the plan exactly as written — step order, verifications, and commits included. When a step is unclear or a verification fails, stop and ask; never guess your way past a blocker.**

## When to Use
- An implementation plan exists and inline execution (not subagents) was chosen
- Resuming a partially executed plan after a blocker was resolved
- A human hands you a plan file and asks you to run it
- **Never skip when:** the plan was produced by the blueprint skill — it was designed to be executed step-by-step, and skipping steps breaks the TDD rhythm and commit trail

## Process

### Phase 1: Load and Review
1. Read the plan file in full
   - Announce: "I'm using the execute-plan skill to run this plan."
2. Review the plan critically before touching anything
   - Are all referenced files, types, and functions traceable within the plan?
   - Are there any steps that are ambiguous or impossible to verify?
   - Does the first task have a clear starting point?
3. If concerns exist: raise them with the user before starting — list each concern specifically
4. If no concerns: proceed to Phase 2

### Phase 2: Environment Setup
1. Confirm you are NOT on main or master
   - If you are, stop and request an isolated branch or worktree before continuing
   - Never begin implementation on a protected branch without explicit user consent
2. Verify the environment is ready: dependencies installed, tests run clean before changes

### Phase 3: Execute Tasks
1. Work through tasks in plan order — do not skip or reorder
2. For each task:
   - Mark it in-progress in your task tracker
   - Execute each step exactly as written
   - Run every verification command specified and confirm the expected output
   - If a step says "Expected: PASS" and the result is FAIL, stop — do not continue to the next step
   - Commit at each checkpoint specified in the plan
   - Mark the task complete only after all steps and verifications pass
3. Checkpoint after every task: confirm the codebase is in a clean, passing state before moving on

### Phase 4: Blocker Protocol
1. Stop executing immediately when any of the following occur:
   - A dependency is missing and cannot be installed without guidance
   - A test fails in a way the plan did not anticipate
   - An instruction references something that does not exist
   - You have run the same verification three times without passing
   - You are unsure whether proceeding is safe
2. Report the blocker with specifics:
   - Which task and step
   - What you tried
   - What the actual output was vs. what was expected
   - What you need to continue
3. Wait for guidance — do not improvise a workaround

### Phase 5: Rollback When Needed
1. If a task cannot be completed and the partial changes would leave the codebase broken:
   - Revert uncommitted changes for that task only: `git restore .`
   - If committed changes need unwinding, identify the exact commits and propose a revert to the user — do not run destructive git commands without approval
2. The goal is always a clean, passing state — never leave the branch in a broken condition between sessions

### Phase 6: Completion and Hand-off
1. After all tasks are marked complete and all verifications pass:
   - Run the full test suite one final time and confirm it is green
2. Announce: "I'm using the finish-branch skill to complete this work."
3. Invoke the `finish-branch` skill to handle test verification, PR prep, and merge readiness

## Red Flags — Stop Immediately
- You are about to skip a verification step because "it's probably fine"
- You are modifying a file not mentioned in the current task
- A test that was passing before your change is now failing and you do not know why
- You have been guessing at what a step means for more than one attempt
- You are on main or master and the plan did not explicitly authorise it
- A commit would bundle unrelated changes from multiple tasks

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "The verification is redundant, I can see the code is correct" | Verification catches environment issues, import errors, and integration failures that reading code cannot |
| "I'll commit everything at the end to keep it tidy" | One large commit makes failure isolation impossible and loses the audit trail the plan was built to create |
| "The step is unclear but I know what it means" | If it is unclear to you, it will be unclear in review — stop and get it clarified in the plan |
| "I'll fix the failing test by adjusting the assertion" | Adjusting assertions to match wrong output hides bugs; investigate the production code first |
| "Skipping this task is fine, it's just scaffolding" | Every task in the plan exists because it was needed — skipping creates gaps the reviewer will find |
| "I'll handle the rollback concern after I finish this task" | A broken intermediate state corrupts subsequent tasks; clean up before moving forward |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1. Load & Review | Read plan, identify concerns | No open questions; user has cleared any concerns |
| 2. Environment | Confirm branch, clean baseline | Tests pass before any changes |
| 3. Execute | Step-by-step with verifications | All tasks marked complete, all checks green |
| 4. Blockers | Stop, report specifics, wait | User provides resolution |
| 5. Rollback | Revert broken partials | Branch returns to a clean, passing state |
| 6. Hand-off | Full suite green, invoke finish-branch | Branch is merge-ready |
