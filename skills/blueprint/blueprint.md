---
name: blueprint
description: Use when you have an approved spec and need a step-by-step implementation plan. Produces a complete no-placeholders plan that any skilled developer can execute.
---

# Blueprint

## The Law
**Ship the plan or skip the plan — there is no middle ground. If a step cannot be followed without guessing, it is a hole, not a step. Name the file. Show the code. Give the command. State the expected output.**

## When to Use
- An approved spec or requirements document exists
- Implementation is about to start
- Handing work to a subagent, another developer, or future session
- Multi-task features where execution order and file ownership must be explicit
- **Never skip when:** the work touches more than one file — unplanned changes cause tangled commits

## Process

### Phase 1: Scope Check
1. Read the spec in full before writing anything
2. If the spec covers multiple independent subsystems, stop and raise it
3. Announce: "I'm using the blueprint skill to write the implementation plan"

### Phase 2: Map File Structure
1. List every file that will be created or modified
2. State what each file owns
3. One clear responsibility per file
4. Follow existing codebase patterns

### Phase 3: Write the Plan
1. Use this header:

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** Use the execute-plan skill to implement task-by-task.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2–3 sentences on the overall approach]

**Tech Stack:** [Key libraries and tools]

---
```

2. Write tasks in order — earlier tasks must not reference types/functions defined in later tasks
3. Each task:
   - Has clear acceptance criteria
   - Produces working, testable software on its own
   - Lists exact file paths
   - Shows code examples or commands
   - States expected output

### Phase 4: Order Tasks for Dependencies
1. List tasks that have no dependencies first
2. List tasks that depend on prior tasks second
3. Never reference a file, function, or type before it's defined

## Red Flags — Stop Immediately

- A task says "implement X" without showing how — write the how
- A task references a function/type that won't exist until a later task — reorder
- A task will take longer than a single hour to implement — break it into smaller tasks
- The plan is more than 1 page per working day of effort — some tasks can be parallelized
- A task has no clear acceptance criteria — define what "done" means

## Common Rationalizations

| Excuse | Why It's Wrong |
|--------|---|
| "The developer can figure out the details" | Not if you don't know what details matter. Specifics prevent misunderstanding. |
| "This plan is too detailed" | Detailed plans take less total time. Vague plans cause rework. |
| "We'll adjust the plan as we go" | Adjustments are fine. But start with specifics so adjustments are small tweaks, not rewrites. |
| "I don't know enough about the codebase to plan file structure" | Learn the codebase first. Planning forces you to understand it. |
| "Let's just code and commit when we're done" | Commits without a plan create mess. A plan produces clean commits. |

## Quick Reference

| Phase | Core Action | Done When |
|-------|---|---|
| **Scope** | Read spec; identify subsystems; announce | Scope is single and clear |
| **Map** | List files; state ownership; follow patterns | File structure is explicit |
| **Plan** | Write tasks with code, commands, expected output | Every task shows exactly how |
| **Order** | Reorder so dependencies are satisfied | No task references undefined code |
