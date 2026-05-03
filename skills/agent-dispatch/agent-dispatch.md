---
name: agent-dispatch
description: Use when coordinating parallel agents — define domain boundaries, dispatch agents with domain-specific context, aggregate results, and manage handoff points. Prevents agents from duplicating work.
---

# Agent Dispatch

## The Law
**Before dispatching agents, define their domains clearly. Each agent owns one problem space and stops at the boundary. Overlapping domains cause duplicate work; gaps cause missed work.**

## When to Use
- A large task can be parallelized into independent subdomains
- Multiple agents can work on different pieces simultaneously
- You need to coordinate results back into one coherent outcome
- **Never skip when:** agent work will be merged into a single codebase — coordination prevents conflicts

## Process

### Phase 1: Define Domains
1. Decompose the task into independent subdomains:
   - Each domain has one clear owner (the agent)
   - Domains do not overlap
   - Each domain can be completed independently
2. For each domain, state:
   - What is in scope
   - What is out of scope
   - What the agent will deliver

### Phase 2: Brief Each Agent
1. For each agent, write a brief:
   ```
   AGENT BRIEF: [Agent Name]
   Domain: [What this agent owns]
   Context: [Background they need to know]
   Deliverable: [What they will produce]
   Constraints: [What they cannot do]
   Dependencies: [What must be done first]
   ```
2. Include exact requirements and acceptance criteria

### Phase 3: Dispatch Agents
1. Send each agent the brief
2. Agents work in parallel
3. Track which agent owns which domain

### Phase 4: Aggregate Results
1. Collect deliverables from each agent
2. Merge them into the final outcome
3. Verify no gaps or overlaps
4. Fix integration issues

### Phase 5: Record Outcomes
1. Document what each agent produced
2. Note any blockers or deviations
3. Update the task list

## Red Flags — Stop Immediately

- Agent domains overlap — redefine boundaries
- An agent is waiting on another agent's work — reorder dependencies or combine agents
- Agent will produce code that conflicts with another agent's code — they're in the same domain, reassign
- A requirement is not owned by any agent — add it to a domain or create a new agent
- Agent briefs are vague — make them specific

## Common Rationalizations

| Excuse | Why It's Wrong |
|--------|---|
| "We can figure out domains as we go" | Without domains, agents duplicate work. Define them upfront. |
| "One agent can do everything faster" | True for small tasks. For large tasks, parallelization is faster and cheaper. |
| "Agents can communicate as needed" | Agents communicating mid-task creates serialization. Define domains to prevent it. |
| "This task can't be parallelized" | Almost every task can be. You just need to find the domain boundaries. |

## Quick Reference

| Phase | Core Action | Done When |
|-------|---|---|
| **Define** | Decompose task; state scope/out-of-scope for each agent | Domains do not overlap, no gaps |
| **Brief** | Write agent briefs with context, deliverable, constraints | Each agent has explicit requirements |
| **Dispatch** | Send briefs; agents work in parallel | Agents are working independently |
| **Aggregate** | Collect deliverables; merge; verify | Final outcome is integrated and correct |
| **Record** | Document what each agent produced; note blockers | Outcomes are recorded |
