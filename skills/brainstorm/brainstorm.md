---
name: brainstorm
description: Use when you have an idea but no design yet. Ask clarifying questions one at a time, propose 2-3 approaches, present designs in sections, get approval before moving to implementation planning.
---

# Brainstorm

## The Law
**Do NOT invoke any implementation skill, write code, scaffold a project, or take implementation action until you have presented a design and the user has approved it. This applies to EVERY project regardless of perceived simplicity.**

## When to Use
- You have an idea or requirement but no specification yet
- The approach is unclear or multiple valid paths exist
- Architectural decisions are needed before coding
- **Never skip when:** the task will touch more than one file — unplanned multi-file changes cause tangled code

## Process

### Phase 1: Explore Project Context
1. Check files, docs, and recent work
2. Understand the existing codebase patterns and constraints

### Phase 2: Ask Clarifying Questions
1. Ask one question at a time to understand purpose, constraints, success criteria
2. Prefer multiple-choice questions when possible
3. Focus on understanding what you're building, not how yet

### Phase 3: Propose 2-3 Approaches
1. Present different approaches with trade-offs
2. Lead with your recommended option and reasoning
3. Help the user understand the costs and benefits of each

### Phase 4: Present Design
1. Once you understand requirements, present the design in sections
2. Scale each section to its complexity (a few sentences for simple, 200-300 words for nuanced)
3. Ask after each section whether it looks right
4. Cover: architecture, components, data flow, error handling, testing
5. Go back and clarify if something doesn't make sense

### Phase 5: Get User Approval
1. Once the user approves the design, proceed to implementation planning
2. If they request changes, update the design and re-present
3. Do NOT proceed to implementation until design is approved

### Phase 6: Document Design
1. Write the validated design to a spec file
2. Commit it to version control
3. Ask the user to review the written spec before proceeding

### Phase 7: Invoke Implementation Planning
1. Once the spec is approved, invoke the writing-plans skill
2. Do NOT invoke any other implementation skill

## Red Flags — Stop Immediately

- You are about to code without presenting a design — stop and brainstorm first
- The user describes multiple independent subsystems — decompose into separate projects first
- You are skipping clarifying questions because the task "looks simple" — simple projects hide unexamined assumptions
- You have been asking clarifying questions for more than 5 exchanges — you have enough information, propose approaches now
- You are proposing 5+ approaches — narrow to 2-3 with the strongest trade-offs

## Common Rationalizations

| Excuse | Why It's Wrong |
|--------|---|
| "This is just a tiny change, we don't need a design" | Tiny changes compound. A good design prevents rework. A bad design costs more to fix than to get right upfront. |
| "I already know what the user wants, let's skip to coding" | You probably don't know what they want. Clarifying questions prevent wasted work. |
| "The design will emerge as we code" | Emergent design causes refactoring and rework. Planned design prevents it. |
| "We don't have time for design" | Skipping design makes the timeline longer, not shorter. |
| "I'll just implement multiple approaches and show the user" | Wasted implementation time. User feedback on a design is faster and cheaper. |

## Quick Reference

| Phase | Core Action | Done When |
|-------|---|---|
| **Explore** | Read project context; understand patterns | You know the codebase landscape |
| **Ask** | One question at a time; understand purpose/constraints | Requirements are clear |
| **Propose** | 2-3 approaches with trade-offs | User understands the options |
| **Design** | Present architecture, components, data flow, error handling, tests | User approves each section |
| **Approve** | Get user sign-off on design | Approval is explicit |
| **Document** | Write spec; commit; ask user to review written spec | Spec is approved in writing |
| **Plan** | Invoke writing-plans skill | Implementation plan is generated |
