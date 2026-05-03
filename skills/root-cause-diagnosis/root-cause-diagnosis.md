---
name: root-cause-diagnosis
description: Use for system-level issues — symptom-to-root-cause tracing using logs, environment state, and hypothesis testing. Diagnostic tooling focused (bash, config reads, no file modifications).
---

# Root Cause Diagnosis

## The Law
**Move from reported symptom to confirmed root cause with traceable evidence, not guesses. Every hypothesis must be ranked, and each ranked hypothesis must be tested with a command or check that confirms or eliminates it.**

## When to Use
- A production service is down or degraded
- Errors appear in logs but the cause is not obvious
- The issue is system-wide (multiple services, infrastructure)
- Environment-specific failures (works locally, fails in staging or prod)
- **Never skip when:** the issue is blocking users or prod is down — evidence-based diagnosis prevents wasted time on wrong fixes

## Process

### Phase 1: Symptom Statement

Before touching any tools, restate the problem:

```
SYMPTOM
Reported: <exact error message or behaviour>
Context:  <where it happens — endpoint, function, UI state, environment>
Onset:    <when it started — after a deploy, always, intermittent?>
```

If onset is unknown, note that explicitly — it changes the search strategy.

### Phase 2: Environment Snapshot

Gather system context. Run each command and record the output verbatim.

```bash
# Running processes (look for crashes, restart loops)
ps aux | grep -v grep | grep -E "(node|python|ruby|java|go|php|nginx|postgres|redis|mysql)"

# Network state (look for open ports, listening services)
netstat -tlnp 2>/dev/null || ss -tlnp

# Environment variables (redact secrets; note version mismatches)
env | grep -E "(DATABASE|REDIS|API|SECRET|PORT|HOST|NODE_ENV|PYTHON_ENV|RAILS_ENV)" | sed 's/=.*/=<redacted>/'

# Dependency versions
cat package.json | grep -E "\"version\"|\"dependencies\"" || cat requirements.txt | head -20
```

Record every output exactly — environment mismatches are often the root cause.

### Phase 3: Log Excavation

Pull the most recent error-bearing lines from logs. Prefer structured logs over raw stdout.

```bash
# Common log paths — adapt to your stack
tail -n 100 /var/log/app/error.log 2>/dev/null || true
tail -n 100 ~/.pm2/logs/*-error.log 2>/dev/null || true
journalctl -u <service> -n 100 --no-pager 2>/dev/null || true
grep -i "error\|exception\|fatal" /var/log/app/*.log | tail -50
```

For each error line found, extract:
- **Timestamp** — when did this occur relative to the symptom?
- **Stack frame** — which file and line is the top of the trace?
- **Message** — verbatim, not paraphrased

### Phase 4: Hypothesis Ranking

List candidate causes, ranked by probability:

```
HYPOTHESIS RANK
1. [HIGH]  <cause> — evidence: <what supports this>
2. [MED]   <cause> — evidence: <what supports this>
3. [LOW]   <cause> — evidence: <why this is worth noting but less likely>
```

Rank by: (a) supported by log evidence, (b) recent code or config change, (c) known failure mode for this stack.

Do not list more than 5 hypotheses.

### Phase 5: Targeted Testing

For each HIGH hypothesis, state a test that would confirm or eliminate it:

```
TEST FOR HYPOTHESIS 1
Command: <exact command to run>
Confirms if: <what output means it's the cause>
Eliminates if: <what output rules it out>
```

Run the test. Report the result. If a hypothesis is eliminated, cross it off and move to the next.

### Phase 6: Root Cause Statement

Once a hypothesis is confirmed, deliver:

```
ROOT CAUSE
Chain:     <A caused B which caused C — the symptom>
Evidence:  <log lines, command output that proves this>
Confidence: HIGH | MEDIUM
If MEDIUM: <what would make this HIGH>

FIX DIRECTION: <one sentence on the type of fix needed — not the full implementation>
REGRESSION RISK: <what else could break if fixed incorrectly>
```

## Red Flags — Stop Immediately

- You propose a fix before confirming the root cause — stop and backtrack
- You skipped Phase 2 (environment snapshot) — environment is evidence
- You paraphrased log output instead of quoting it — paraphrasing hides details
- You labeled a guess as HIGH confidence — unconfirmed hypotheses are LOW or MED
- No timestamp correlation between events — are you looking at the right logs?

## Common Rationalizations

| Excuse | Why It's Wrong |
|--------|---|
| "The logs don't show anything obvious, so I'll guess" | Logs are evidence. If you don't see the cause, you haven't dug deep enough. Check timestamps, multiple files, older logs. |
| "This has happened before and the fix was X, so X is the cause again" | Different events can have different causes. Precedent is a starting hypothesis, not proof. Test it. |
| "The environment looks fine, we can skip that check" | Environment mismatches are the most common production issues. Never skip Phase 2. |
| "I'll just restart the service and see if it fixes it" | Restart masks the problem. You need to understand the cause before restarting, or it will happen again immediately. |
| "The fix is more important than understanding the cause" | Understanding the cause prevents wasted fixes. A fix built on guesses causes more downtime later. |

## Quick Reference

| Phase | Core Action | Done When |
|-------|---|---|
| **Symptom** | State reported, context, onset | Symptom statement is clear and concrete |
| **Environment** | Run commands; record all output verbatim | All 4 environment checks completed, output recorded |
| **Logs** | Extract recent errors; note timestamps and stack frames | Top 3 error patterns identified with timestamps |
| **Hypotheses** | Rank candidates by evidence, recency, known failure modes | 2–5 hypotheses ranked by probability |
| **Testing** | Write test for each HIGH hypothesis; run and report | All HIGH hypotheses tested, at least one confirmed |
| **Root Cause** | Chain evidence → symptom; state fix direction and risk | Root cause chain is explicit, confidence stated, risk assessed |
