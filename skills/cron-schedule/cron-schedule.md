---
name: cron-schedule
description: Set up a cron-scheduled Claude Code agent task, manage existing schedules, and verify execution.
---

# Schedule Task

## The Law
**Every scheduled agent must have an explicit schedule expression, a concrete prompt describing exactly what to do, and a verification step that confirms it ran correctly — never schedule a vague task and hope for the best.**

## When to Use
- A recurring task (daily report, weekly cleanup, hourly health check) needs to run without manual intervention
- A one-time future action needs to be queued and tracked
- An existing schedule needs to be inspected, updated, or cancelled
- **Never skip when:** the task touches production data or external systems (those need an explicit prompt and output verification), or when the schedule involves multiple steps (each step must be described in the prompt, not assumed)

## Process

### Phase 1: Define the Task
1. Write out the task in plain language before configuring anything:
   - What should happen?
   - What inputs does it need (file paths, API endpoints, environment variables)?
   - What is the expected output or side effect?
   - Who should be notified if it fails?
2. Convert the plain-language description into a concrete prompt. Vague prompts produce vague results:
   - Bad: "Check the database"
   - Good: "Connect to the production Postgres database at DATABASE_URL, count rows in the `orders` table where `created_at > now() - interval '1 day'`, and write the result to `/tmp/daily-order-count.txt`"

### Phase 2: Choose the Schedule Expression
1. Write the cron expression and verify it before saving:

   | Expression | Meaning |
   |---|---|
   | `0 9 * * 1-5` | 09:00 UTC, Monday through Friday |
   | `0 */4 * * *` | Every 4 hours |
   | `30 7 * * 1` | Every Monday at 07:30 UTC |
   | `0 0 1 * *` | First of each month at midnight UTC |
   | `*/15 * * * *` | Every 15 minutes |

2. Validate the expression at [crontab.guru](https://crontab.guru) or with:
   ```bash
   # Python validation
   from croniter import croniter
   cron = croniter("0 9 * * 1-5")
   print(cron.get_next(float))  # next scheduled run as timestamp
   ```
3. Note the timezone — cron expressions are evaluated in UTC by default. Adjust if the task is time-sensitive to a local timezone.

### Phase 3: Create the Schedule
Use the `schedule` skill to create a new routine:
```
/schedule create
  name: daily-order-count
  prompt: >
    Connect to the production Postgres database at $DATABASE_URL.
    Count rows in the orders table where created_at > now() - interval '1 day'.
    Write the result to /tmp/daily-order-count.txt in the format:
    "YYYY-MM-DD: N orders"
  schedule: "0 9 * * 1-5"
```

Or via the CLI if the runner supports it:
```bash
claude schedule create \
  --name "daily-order-count" \
  --prompt "Count yesterday's orders and write to /tmp/daily-order-count.txt" \
  --cron "0 9 * * 1-5"
```

### Phase 4: Manage Existing Schedules

**List all schedules:**
```bash
claude schedule list
# or
/schedule list
```

**Inspect a schedule:**
```bash
claude schedule show daily-order-count
```

**Update a schedule's prompt or cron expression:**
```bash
claude schedule update daily-order-count --cron "0 8 * * 1-5"
```

**Pause a schedule without deleting it:**
```bash
claude schedule pause daily-order-count
```

**Resume a paused schedule:**
```bash
claude schedule resume daily-order-count
```

**Delete a schedule permanently:**
```bash
claude schedule delete daily-order-count
```

### Phase 5: Trigger a Manual Test Run
1. Before trusting the schedule, run it once manually to confirm the prompt produces the expected result:
   ```bash
   claude schedule run daily-order-count
   # or
   /schedule run daily-order-count
   ```
2. Inspect the output — confirm the file was written, the API call succeeded, or whatever the task was supposed to do.
3. If the output is wrong, fix the prompt (Phase 1) and re-test before leaving it on the schedule.

### Phase 6: Verify Ongoing Execution
1. After the first scheduled run, check the execution log:
   ```bash
   claude schedule logs daily-order-count --limit 10
   ```
2. Set up an alert or notification for failures — a silent failure on a scheduled task may go unnoticed for days:
   ```
   # Add to prompt:
   "If any step fails, write the error to /tmp/daily-order-count-error.txt
   and exit with a non-zero status."
   ```
3. Review the log weekly for the first month to confirm the task is behaving as expected.

## Red Flags — Stop Immediately
- The prompt contains placeholder text like "do the usual thing" or "check everything"
- The cron expression has not been validated — an off-by-one in a cron field means it never runs (or runs constantly)
- The task writes to a production database without a dry-run or confirmation mechanism in the prompt
- No manual test run was performed before trusting the schedule
- The schedule has been running for more than a week with no log review
- A task that sends emails or external notifications has no rate-limit guard in its prompt

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "The prompt can be vague — Claude will figure it out" | Scheduled runs have no human in the loop to correct misunderstandings |
| "I'll check the logs eventually" | Silent failures on schedules can cause data drift for days before anyone notices |
| "The cron expression looks right" | "Looks right" is not the same as validated; one wrong field inverts the schedule |
| "A manual test is unnecessary — it's just a read-only task" | Read-only tasks can still fail or produce wrong output; test it anyway |
| "I can always delete the schedule if it causes problems" | Deleting after the fact does not undo damage already done by a misfiring prompt |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1. Define | Write concrete prompt with explicit inputs and expected output | No ambiguity in what the task does |
| 2. Schedule | Choose and validate cron expression | Expression confirmed with next-run time |
| 3. Create | `claude schedule create` or `/schedule create` | Schedule listed in `schedule list` |
| 4. Manage | Use list / show / update / pause / delete as needed | Schedule state matches intent |
| 5. Test | `claude schedule run` manually | Output matches expected result |
| 6. Verify | Review logs after first scheduled run | Execution confirmed; failure alerting in place |
