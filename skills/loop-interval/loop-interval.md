---
name: loop-interval
description: Set up a recurring task that runs a prompt or slash command on a fixed interval or at a self-paced cadence until cancelled.
---

# Run on Interval

## The Law
**Every interval loop must have an explicit stopping condition known before the first iteration starts — an open-ended loop with no exit is not a loop, it is a runaway process.**

## When to Use
- Polling for a build, deploy, or test run to complete
- Running a recurring check (lint, review, security scan) on a fixed schedule during a long session
- Watching a file or endpoint for a change and acting when it arrives
- **Never skip when:** the task description uses "keep checking", "every N minutes", or "until X happens" — those phrases require a loop, not a one-shot command

## Process

### Phase 1: Define the Loop
Establish four parameters before starting:

| Parameter | Question to Answer |
|-----------|-------------------|
| Prompt or command | What runs each iteration? |
| Interval | How long to wait between iterations? (or "self-paced" if the task sets its own pace) |
| Stop condition | What result, state, or event ends the loop? |
| Max iterations | What is the hard ceiling on iterations, even if the stop condition never fires? |

If the user has not specified a stop condition, ask before starting. An interval loop without a stop condition must not be started.

### Phase 2: Choose the Interval Mode

**Fixed interval** — wait a set time between each run:
```
/loop 5m /review
/loop 2m /security-scan
/loop 10m "check if the deploy succeeded: run `curl -s https://api.example.com/health`"
```

**Self-paced** — the model decides when to run the next iteration based on the task:
```
/loop /babysit-prs
/loop "monitor the test suite and report when it goes green"
```

Use self-paced when each iteration has variable duration (e.g. waiting for a network response). Use fixed interval when the cadence must be predictable regardless of task length.

### Phase 3: Write the Loop Prompt
A good loop prompt answers three questions in one sentence:
1. What to check or do
2. What to report each iteration
3. What to do when the stop condition is met

Template:
```
Every <interval>, <action>. Report <what to summarize>. Stop when <condition> and then <final action>.
```

Examples:

Simple polling loop:
```
Every 3 minutes, run `curl -s https://api.example.com/health | jq .status` and report the value. Stop when the status is "ok" and summarize how many attempts it took.
```

Python test watcher:
```
Every 2 minutes, run `pytest -x -q 2>&1 | tail -5` and report the exit code and last line. Stop when the exit code is 0 and report which tests were still failing on the previous iteration.
```

Self-paced PR monitor:
```
Check open PRs with `gh pr list --state open`. For each PR older than 24 hours with no reviewer assigned, post a comment asking for a review. Repeat after a 15-minute wait. Stop after 3 iterations or when all PRs have reviewers assigned.
```

### Phase 4: Set the Hard Ceiling
Always define a maximum iteration count to prevent runaway loops:

```
Run at most 20 iterations. If the stop condition has not fired by then, report the last observed state and stop.
```

For time-based ceilings:
```
Stop after 2 hours regardless of whether the condition is met.
```

### Phase 5: Monitor and Cancel
1. At each iteration, report:
   - Iteration number and elapsed time
   - Result of the action
   - Whether the stop condition was evaluated and what it found
2. If the stop condition fires: complete the final action, report the outcome, and stop.
3. If the max iteration count is hit: report the last state and stop.
4. The user can cancel the loop at any time by interrupting the session (`Ctrl+C`) or by sending a message that includes the word "stop".

## Red Flags — Stop Immediately
- No stop condition was defined and you are about to start the first iteration anyway
- The interval is less than 30 seconds — that cadence will exhaust context or hit API rate limits
- The loop prompt does not include what to do when the stop condition fires
- The max iteration count was not set

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "I'll stop it manually when it's done" | Manual cancellation requires the user to watch — that defeats the purpose of a loop |
| "The stop condition is obvious from context" | Obvious is not explicit — state it in the prompt |
| "5-second intervals will catch the event faster" | Sub-minute intervals burn context and can trigger rate limits; use webhooks instead |
| "No ceiling needed, it will stop naturally" | If the stop condition never fires, the loop runs forever |
| "Self-paced is fine for everything" | Fixed intervals give predictable cadence when timing matters |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1. Define | Set prompt, interval, stop condition, max iterations | All four parameters confirmed before starting |
| 2. Mode | Choose fixed interval or self-paced | Mode selected and justified |
| 3. Prompt | Write loop prompt covering action, report, and exit | Prompt answers all three questions |
| 4. Ceiling | State max iterations or time limit | Hard ceiling set |
| 5. Monitor | Report each iteration; stop on condition or ceiling | Loop ends with a clear final state summary |
