---
name: incident-lead
description: Lead a structured incident response from first alert through resolution and post-mortem hand-off.
---

# Incident Lead

## The Law
**Declare severity in the first two minutes, open a timeline log immediately, and keep communication flowing to stakeholders every 15 minutes until the incident is resolved — never go dark.**

## When to Use
- A production system is degraded, unavailable, or behaving incorrectly
- An alert fires that has real user or business impact
- You are the first engineer to notice something that needs coordinated response
- **Never skip when:** you think the issue will "probably fix itself" (document it anyway — if it does not, you will need the timeline), or when only a small percentage of users are affected (small blast radius today can become full outage in minutes)

## Severity Classification

Assign severity at declaration and revisit it if conditions change.

| Level | Definition | Response Target |
|-------|------------|-----------------|
| SEV-1 | Full outage or data loss in progress | Immediate; all-hands |
| SEV-2 | Significant degradation or partial outage | Within 5 minutes |
| SEV-3 | Minor degradation; workaround exists | Within 30 minutes |
| SEV-4 | Cosmetic or low-impact; no data risk | Next business day |

## Process

### Phase 1: Declare (0–2 min)
1. Classify severity using the table above.
2. Open a dedicated incident channel or thread: `#inc-YYYY-MM-DD-short-description`.
3. Post the declaration message immediately (see template below).
4. Start the timeline log — a running bullet list with timestamps.
5. Page the on-call if SEV-1 or SEV-2.

**Declaration template:**
```
INCIDENT DECLARED — SEV-{N}
Time:          {HH:MM UTC}
Summary:       {One sentence describing what is wrong}
Impact:        {Who/what is affected and estimated scale}
Incident lead: {name}
Bridge/channel: {link}
Next update:   {15 min from now}
```

### Phase 2: Assemble and Orient (2–10 min)
1. Confirm who is in the bridge — at minimum: incident lead, an engineer with production access, and a communications owner.
2. State the known facts only — do not speculate on root cause yet.
3. Assign explicit roles:
   - **Lead:** owns the timeline, coordinates work, decides escalation
   - **Investigator(s):** dig into logs, metrics, recent deploys
   - **Comms owner:** posts stakeholder updates, manages the status page
4. Pull the first data points:
   ```bash
   # Check recent deploys
   git log --oneline -20

   # Check error rates via pod logs
   kubectl logs -n production -l app=api --since=30m | tail -100

   # Share the metrics dashboard URL in the channel
   ```

### Phase 3: Investigate (ongoing)
1. Log every finding and action in the timeline as it happens — do not reconstruct from memory later.
2. Work hypotheses, not hunches: state what you expect to see if a theory is correct, then check.
3. Check the four most common causes first:
   - Recent deploy or config change
   - Dependency failure (database, external API, cache)
   - Traffic spike or quota exhaustion
   - Infrastructure failure (host, region, network)
4. Share findings in the channel as they arrive — even negative results ("checked DB — query times normal").
5. Post a stakeholder update every 15 minutes regardless of progress:
   ```
   UPDATE {HH:MM UTC} — SEV-{N} still active
   Status:      {Investigating / Mitigating / Monitoring}
   Finding:     {What has been learned}
   Next action: {What is being tried now}
   Next update: {15 min from now}
   ```

### Phase 4: Mitigate
1. Prefer the fastest safe mitigation over the most elegant fix — roll back a deploy, redirect traffic, disable a feature flag.
2. Announce the mitigation action in the channel before executing it.
3. Log the action and its result in the timeline.
4. Monitor key metrics for at least 5 minutes after each mitigation attempt before declaring success.
   ```bash
   # Watch error rate every 30s
   watch -n 30 'curl -s https://metrics.internal/api/errors | jq .rate'

   # Python equivalent
   import time, requests
   while True:
       rate = requests.get("https://metrics.internal/api/errors").json()["rate"]
       print(f"{time.strftime('%H:%M:%S')} error rate: {rate}")
       time.sleep(30)
   ```

### Phase 5: Resolve
1. Declare resolution only when the impact is confirmed gone, not when the fix is deployed.
2. Post the resolution message:
   ```
   RESOLVED — SEV-{N}
   Time:       {HH:MM UTC}
   Duration:   {X hours Y minutes}
   Impact:     {Final scope description}
   Fix applied: {What was done}
   Next step:  Post-mortem by {date}
   ```
3. Update the status page.
4. Thank the responders in the channel.

### Phase 6: Hand Off to Post-Mortem
1. Archive the timeline log to a document before the channel is cleaned up.
2. Assign a post-mortem owner and set a deadline (within 5 business days for SEV-1/2).
3. Post-mortem must contain: timeline, root cause, contributing factors, action items with owners and due dates.

## Red Flags — Stop Immediately
- More than 15 minutes have passed without a stakeholder update
- Root cause is being assumed without evidence — stop guessing and gather data
- A mitigation is being applied without announcement in the channel
- The timeline log has not been updated in the last 10 minutes
- Severity has not been re-evaluated after significant new information
- The bridge has no designated lead — decision authority must be explicit

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "I'll start the timeline once I know what's happening" | The timeline captures the investigation — start it at declaration |
| "It's probably just a blip, not worth declaring" | Declaring a SEV-4 costs 2 minutes; missing a real incident costs hours |
| "We don't need a comms owner, it's minor" | Someone will ask for status; no assigned owner means guaranteed silence |
| "I'll post an update once I have something concrete" | Silence reads as chaos to stakeholders; post progress even without answers |
| "We can skip the post-mortem for small incidents" | Small incidents share root-cause patterns with large ones |
| "Rolling back will disturb other things" | A degraded system is already disturbed — rollback is the safest fast path |
| "We'll figure out severity after investigation" | Severity drives who gets paged; delay means a delayed response |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1. Declare | Severity classified; channel open; declaration posted | Timeline log started |
| 2. Assemble | Roles assigned; initial data pulled | Everyone knows their job |
| 3. Investigate | Hypotheses checked; findings logged | Root cause known or mitigations ready |
| 4. Mitigate | Fastest safe fix applied and announced | Impact confirmed reducing |
| 5. Resolve | Resolution message posted; status page updated | Impact confirmed gone |
| 6. Hand-off | Timeline archived; post-mortem assigned | Owner and deadline confirmed |
