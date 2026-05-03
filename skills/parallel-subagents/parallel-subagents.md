---
name: parallel-subagents
description: Use when executing an implementation plan with independent tasks in the current session, dispatching fresh subagents per task with two-stage review gates
---

# Run Parallel Work

## The Law
**Every task gets a fresh subagent with handcrafted context — never let subagents inherit your session history or skip either review stage.**

## When to Use
- You have a written implementation plan with two or more tasks that can proceed without shared state
- Tasks touch separate files or subsystems and will not interfere with each other
- You want continuous progress in the current session without switching to parallel sessions
- **Never skip when:** the plan has more than one task — even "quick" tasks deserve isolated context and two-stage review

## Process

### Phase 1: Load the Plan
1. Read the plan file once; extract all tasks with their full text and any shared context notes
2. Record the list with TodoWrite so progress is visible throughout the session
3. Do not give subagents the plan file path — provide the full task text directly

### Phase 2: Implement Each Task
1. Choose model tier: cheap model for isolated 1-2 file tasks, standard model for multi-file coordination, top-tier model for design or broad-codebase work
2. Build the implementer prompt: task text, scene-setting context, coding standards, test expectations, expected status code on exit
3. Dispatch implementer subagent and wait
4. Handle the status:
   - `DONE` — move to spec review
   - `DONE_WITH_CONCERNS` — read concerns, address correctness issues before review, note observations
   - `NEEDS_CONTEXT` — supply the missing information and re-dispatch
   - `BLOCKED` — diagnose: add context and retry, escalate model tier, split the task, or surface to the user

**Python async dispatch example:**

```python
import asyncio

async def dispatch_task(task_text: str) -> str:
    """Dispatch a subagent task and return its output."""
    # In a real agentic framework, use the SDK's agent dispatch API.
    # This illustrates the async fan-out pattern.
    result = await agent_sdk.run(prompt=task_text)
    return result

async def run_all_tasks(tasks: list[str]) -> list[str]:
    return await asyncio.gather(*[dispatch_task(t) for t in tasks])
```

**TypeScript dispatch example:**

```typescript
const results = await Promise.all([
  Task("Implement auth module per spec: ...", { context: authTaskText }),
  Task("Add rate-limit middleware per spec: ...", { context: rateLimitTaskText }),
]);
```

### Phase 3: Spec Review
1. Dispatch spec-compliance reviewer with: the original task spec, the commit range or changed files, and a clear pass/fail mandate
2. If reviewer flags gaps: send the same implementer subagent back with specific corrections — do not move forward
3. Re-dispatch spec reviewer until it returns clean

### Phase 4: Code Quality Review
1. Only start this phase after spec review is clean
2. Dispatch code-quality reviewer with the changed files and any project standards
3. If reviewer flags issues: implementer fixes, reviewer re-checks — repeat until approved
4. Mark the task complete in TodoWrite

### Phase 5: Wrap Up
1. After all tasks pass both reviews, dispatch a final reviewer over the full changeset
2. Follow the branch-finishing workflow to prepare for merge

## Red Flags — Stop Immediately
- You are about to let a subagent read the plan file itself instead of receiving the task text
- You are starting code quality review before spec review is confirmed clean
- You are dispatching two implementer subagents for the same task at the same time
- An implementer reported `BLOCKED` and you are retrying with no changes to context, model, or scope
- You are on main/master without explicit user consent
- A reviewer found issues and you are advancing to the next task anyway

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "The spec reviewer is optional for small tasks" | Size does not determine correctness — small tasks drift from spec too |
| "I can do both reviews in one pass" | Spec compliance and code quality are different concerns; conflating them misses both |
| "The implementer self-reviewed, that's enough" | Self-review does not substitute for independent review; run both stages |
| "The tasks are mostly independent, close enough" | "Mostly" means shared files exist — verify before dispatching in parallel |
| "Giving the subagent the plan file saves time" | Subagents read everything in the file, polluting their context; provide only what they need |
| "The blocker is minor, just retry the same model" | If the implementer reported blocked, something must change — context, model, or scope |
| "I'll review quality first since I already have the diff" | Wrong order every time; spec gaps invalidate quality conclusions |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1. Load | Extract all tasks, create TodoWrite | Full task list captured, no plan re-reads needed |
| 2. Implement | Dispatch fresh subagent with handcrafted context | Status is DONE or DONE_WITH_CONCERNS |
| 3. Spec Review | Independent reviewer checks spec compliance | Reviewer returns clean pass |
| 4. Quality Review | Independent reviewer checks code quality | Reviewer approves with no open issues |
| 5. Wrap Up | Final full-changeset review, branch finish | All tasks complete, branch ready for merge |
