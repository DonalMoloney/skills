---
name: codex-rescue
description: Use when a subagent (Codex) is stuck or needs a second implementation attempt. Diagnose the blocker, provide a new approach, and hand back with clear direction.
---

# Codex Rescue

## The Law
**When a subagent stops, do not re-run it with the same prompt. Diagnose why it stopped, provide a different approach, and hand it back with clear recovery direction.**

## When to Use
- A subagent has timed out or hit an error
- A subagent has produced incomplete output
- An implementation approach needs a second pass from a different angle
- **Never skip when:** a subagent is stuck — rescue it rather than giving up

## Process

### Phase 1: Assess the Blocker
1. Read what the subagent produced
2. Identify why it stopped:
   - Out of context window?
   - Hit an error it couldn't recover from?
   - Approach was too complex for one pass?
3. Note what progress was made

### Phase 2: Diagnose Root Cause
1. Was the task too large?
   - Split it into smaller subtasks
2. Was the approach wrong?
   - Propose a different strategy
3. Was the tooling insufficient?
   - Provide different tools or context
4. Output:
   ```
   RESCUE DIAGNOSIS
   Blocker: <what stopped the agent>
   Root cause: <why it stopped>
   Progress: <what was accomplished>
   Next approach: <different strategy to unblock>
   ```

### Phase 3: Provide New Direction
1. Reframe the task with:
   - Different approach (if the first failed)
   - Smaller scope (if too large)
   - More specific guidance (if vague)
   - Required context it lacked (files, patterns, examples)

### Phase 4: Hand Back with Clear Checkpoint
1. Specify exactly where the subagent should resume:
   - Which file to start with?
   - What decision point did it stop at?
   - What should it prioritize differently?
2. Set a new stopping point (not open-ended)

### Phase 5: Monitor Second Attempt
1. Give the subagent the rescue brief
2. Monitor for progress
3. If stuck again, assess whether the task needs human intervention

## Red Flags — Stop Immediately

- The subagent made no progress at all — the task is misspecified, not stuck code
- The subagent is in a loop (same error repeatedly) — intervene immediately
- The rescue itself has become vague — you don't know what to do either; step back
- More than 2 rescue attempts needed — the task is genuinely too hard for delegation

## Common Rationalizations

| Excuse | Why It's Wrong |
|--------|---|
| "Just re-run the agent with the same prompt" | It will hit the same blocker. Diagnose and change the approach. |
| "The agent failed, so we should do this manually" | Not necessarily. A rescue often unblocks it faster than manual work. |
| "The task is clear enough, the agent just isn't good enough" | If a capable agent is stuck, the task is the problem. Clarify or split it. |
| "We'll figure it out as we go" | Define the next step before handing back. Vague rescues fail again. |

## Quick Reference

| Phase | Core Action | Done When |
|-------|---|---|
| **Assess** | Identify what blocker stopped the agent | Blocker is clear |
| **Diagnose** | Root cause: too large / wrong approach / insufficient tooling | Root cause is named |
| **Reframe** | New approach, smaller scope, or additional context | Rescue brief is written |
| **Checkpoint** | Specify resume point and new stopping point | Subagent knows exactly what's next |
| **Monitor** | Hand back and watch for progress | Second attempt is underway |
